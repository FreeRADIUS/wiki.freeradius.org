# Message Channel

We need a discovery mechanism for queues / threads, and a message exchange

## Messages

* KQ is kqueue
* CAQ is control atomic queue. used for control-plane signalling
* DAQ is data atomic queue, used for data-plane signalling
* ID is an identifier unique to the queue.

_We may not need a control plane queue, if all of the signals can be
handled in the KQ_.

new Q (KQ_n, CAQ, DAQ, ID) -> KQ_w

* this all has to be in one data structure, to fit in the kqueue infrastructure

del Q (ID) -> KQ_w

read data (ID) -> KQ_w

started busy poll (ID) -> KQ_w

stopping busy poll (ID) -> KQ_w

OK to stop stop busy poll (ID) -> KQ_w

*network* discard packets (ID) -> KQ_w

*network* duplicate packet (ID, packet ctx) -> KQ_w

*network* extend lifetime packet (ID, packet ctx) -> KQ_w

## Data structure

KQ_other, CAQ_mine, DAQ_mine

worker thread has RB tree / linked list of requests, so that they can
be cleaned up or sent DUP signals

The request has the incoming queue information.  It needs to be able
to use that to find the outgoing queue information.

```
typedef struct fr_channel_end_t {
	int			kq;
	intptr_t		id;
	int			poll_state;	// signalling, busy-poll, etc.
	int			status;		// alive / zombie / dead
	size_t			num_active	// managed by each end
	fr_atomic_queue_t	*control;
	fr_atomic_queue_t	*data;
} fr_channel_end_t;

typedef struct fr_channel_t {
	intptr_t		id;		// copy for locality

	fr_channel_end_t	*remote;	// allocated and managed by them!

	fr_channel_t		*next;		// for the same ID?  so that the queues can grow

	rbtree_t		*tree;		// for active requests ??

	fr_channel_end_t	local;		// can be allocated here for locality
} fr_channel_t;
```

## Queue specific signalling

### new Q (KQ_n, CAQ, DAQ, ID) -> KQ_w

network thread discovers a worker thread KQ (somehow), and sends it a
signal of an `fr_channel_end_t`.  The worker thread finds / allocates
`fr_channel_t` (with it's own end), and sends a pointer to it's own
`fr_channel_end_t` to the other.

The worker thread tracks it's queues by ID.  There should be ~256 IDs
maximum, one for each CPU on the system.  Which makes lookups O(1).

### del your queue (ID, [CAQ], [DAQ]) -> KQ_n

After a worker thread receives a a new queue, AND when the old queue
is empty, the worker signals the network thread that the old queue can
be deleted.

### del my Q (ID, action) -> KQ_w

Delete my queue, with an associated action.

i.e. discard all packets, or let all packets continue, BUT don't send
the replies anywhere.

Or maybe have the `del my Q()` call supply a function which takes a
`REQUEST`, and does something with it?  I.e. deletes it, or removes
the queue from it.

**Note** the `reply` function MUST be able to take a `NULL` queue from
a request, and then skip the encode routine.  e.g. for accounting
packets, where we probably do want to store the accounting data, but
then not actually send a reply.

### data ready for Q (ID, [message]) -> KQ_w

Signal that data is ready.  Initially sends the data, too.

If there are more too many packet/s of messages, the network thread
sends a message to the worker thread saying "please start busy
polling".  It continues to send messages until it notices that the
destination queue has been marked "busy polling".  Once that happens,
it starts pushing packets onto the atomic queue.

i.e. the atomic queue is only used when busy polling.

The "too many packets/s" metric is hard to judge.  It probably should
be found by comparing the inter-packet latency, with the average
packet processing time.

### starting busy poll (ID) -> KQ_n

Signal the other end that the recipient is busy-polling the Q.  And
that the `data read for Q (ID)` signals should no longer be sent.  No
ACK is necessary.  If any `data read for Q (ID)` signals are received
after this message has been sent, the data is checked as normal.

### stopping busy poll (ID) -> KQ_n

Signal the other end that the recipient is intending to stop
busy-polling the Q.  The other end should reply with an `ok to stop
busy poll(ID)`

### ok to stop busy poll(ID) -> KQ_w

Signal the other end that it's OK to stop busy polling the Q.  After
this point, the originator will signal the recipient every time data
is ready.

Since we have the queue, we don't really need this as a signal.  We
can just busy-poll until we see the ACK in the `fr_channel_end_t`.
Which doesn't even have to be atomic, because only one thread is ever
writing to it.  The other thread is only reading, and is OK with
reading stale information.

## Data Signalling

i.e. messages

### New packet(ctx)

Packets have priorities (0..65535).  Packets are put into a queue in
time order, but with a priority.  The worker thread puts them into a
priority heap, so that low priority packets can be ignored.

There are no "dup" or "conflicting" packet messages.

Instead, the network thread tracks the packet status in it's de-dup
tree (if applicable).  When a request becomes runnable, the worker
thread checks:

1. The exchange status.  if the exchange is dead, the request is
deleted.

2. the network de-dup status.  If the packet there is newer
(timestamp), then the current request is cancelled.

And that status check has to be a callback function.  Most of the time
it can be an internal one.  For accounting, we need a different one.

## Tracking table

The tracking table is used for RADIUS auth / acct packets.  For auth,
it also does dedup.  For acct, it tracks conflicting packets.  i.e. if
we get a new packet in while the old one is live, we know to *not*
send the old reply.

We don't need to track the file descriptor in the entry, because the
tracking table is unique to each FD.

The key thing to note is that we use connected UDP sockets.  Protocols
like DHCP do not need de-dup or packet tracking (in input), so they do
not have tracking tables or de-dup trees.

The tracking tables are protocol-specific, and belong in the
protocol-specific directories.  We probably only need to export a
`fr_time_t const*` pointer for old / new packets.  Any other state is
not atomic, and gets difficult to manage safely without mutexes.