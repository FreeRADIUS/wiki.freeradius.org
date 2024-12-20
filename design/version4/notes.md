# Miscellaneous Notes

To be collated and organized later.

## Dynamic Clients

If we have dynamic clients, then the "raw" packet should get sent from the
network thread to a worker thread.  In v3, the network thread process it.

Passing the raw packet to the worker thread allows the worker thread
to fully decode the packet, and have dynamic client rules based on
packet contents.  The worker thread can then send a message back to
the network thread, saying "yes, add this client".

Even better, if we have connected UDP sockets, the dynamic client
tracking can be done *per connection*.  While this means more clients
are created and destroyed, it also allows for multiple clients behind
a NAT, each with a different shared secret.

In general (the hope goes) if a client behind a NAT continuously sends
packets from one source port, the NAT should maintain a mapping for
that client.  The server then sees a consistent NAT src IP/port for
those packets, i.e. a connection.

A different client behind the same NAT will be mapped to a different
NAT port, and will therefore be a different connection.  As the
connection expires, so does the dynamic client.

Each "dynamic client" configuration may then be marked with a "NAT"
flag.  If that particular client is a NAT, then other clients hidden
behind that NAT should use connected UDP sockets.

Similar arguments apply for TCP.

## State and ongoing packets

We likely need to handle State attributes for servers other than
ourselves.

For example, if we're a proxy, we probably want to track ongoing
conversations, so that the bulk of the "should we proxy?" decision is
done on the first packet, and then cached in the `session-state` list.
Subsequent packets can then be processed through a virtual server
which is specific to ongoing sessions, instead of brute-forcing them
through the same rules every time.

This likely means that we need to have multiple state trees
internally.  One for states we create ourselves.  And `N` more, one
for each Realm.  The idea is that each Realm creates and manages it's
own State, and that (theoretically) States for different Realms can
overlap.

This means that someone has to parse the User-Name, early in the
process.  Maybe the network thread?

It also likely means that this works for RFC 7542 style realms
(`@example.com`).  Other kinds of realms do *not* get this tracking.
Oh well, too bad for them.

## Control socket

i.e. sockets which send a lot of data.

The state machine for the worker thread needs to be fully async.
i.e. the underlying IO such as `cprintf` in `src/main/command.c` has
to be able to `YIELD` and `RESUME`.

The protocols that write lots of data need to be self-clocked with
respect to the network.  They send data (10K, 100K, 1M, etc.) to the
network thread, along with a message of "wake me up when at 50%" (for
example).  The network thread writes the data, and when it reaches the
previously-set mark, notifies the worker thread.

This process means that the worker thread is async, and uses memory
buffers to even out bursts of traffic.  The network thread is async in
network IO, and just reads / writes messages internally.

Or, we could just push the file descriptor to the worker thread, and
have it do the blocking writes.

## Timing and signaling for Queues

When a message goes into a queue, the recipient generally needs to be
signalled.  However, kqueue / select APIs don't work well with
pthread_cond_wait().  So we need a solution.

One solution is kqueue user signals, or self pipes.  The problem with
this approach is that it involves a system call to send the signal,
and a system call to receive it.  Which is huge overhead at high
packet rates.

A better approach for high packet rates is self-clocking.  When a
network thread reads a packet, it also checks the incoming queues from
the worker threads.  If the packets are coming in quickly enough, this
(somewhat delayed) check will add minimal latency.

Even better, for UDP sockets, the worker thread can just send the
reply itself, and then send the message to the network thread.  If the
network thread receives a new packet using the same ID, it will ignore
the message.  Otherwise, it will put the packet into the dedup tree.

The queue reader can put a timer in (say 1ms) so that if it doesn't
get signalled by something else, it still wakes up to service the
timer and check the queues.  If a few timers go off without there
being anything in the queue, it sets a flag in the queue saying
"signal me when there is a new queue entry".  And sets one last timer,
and after that timer fires, goes to sleep.

The last timer fire is there to (mostly) catch race conditions where
the message originator doesn't see the flag, and therefore doesn't
send a signal, even though the recipient is expecting one.

The result of this design is that for high packet rates there is
(perhaps) some small latency added, but not much.  For low packet
rates, there is a small amount of busy polling before the system goes
to sleep.

### A better approach

An even better approach is for the originator to have it's own queue
of outstanding messages which it's sending to a particular
destination.  Then, instead of sending messages, it sends the queue.

i.e. it inserts objects into it's own local queue.  If the queue is
empty, the originator inserts it's local queue into the receipients
inbound xqueue, and then signals the recipient.

If the local queue is non-empty, the originator assumes that the
recipient has already picked up the message, and does not signal the
recipient.

We could also put the messages into a linked list, with head / tail /
cleanup, but that would probably involve a lot more memory thrashing.

The result *should* be massively reduced contention on the recipients
inbound queue.

When the recipient gets a signal, it pulls the originators queue off
of the inbound queue, and caches it locally.  Any messages are then
de-queued by the recipient, directly from the originators queue.  This
design means that the recipient has to keep local references to each
originators queue.  There are cleanup / timing issues associated with
this, but are solvable.

The recipient (if busy) then busy-polls the queues until they're
empty.  At which point, it discards the queue, and waits for a new
signal.  There should be some signal to the caller that it needs to
re-start signalling?

Or does it need to do that?  The only thing is that the recipient has
to *not* check for signals in between the time the queue goes from
1..0, and the time it drops the queue.

Or, the recipient signals the caller via a message that it's dropping
the queue.  e.g. mark messages "busy-looping", then "about to drop the
queue", followed by "acked from the originator", and then it can drop
the queue.

### After some discussion

The signals should be limited to control-plane actions.

* here's a new queue for you
* please check the queue
* I'm going to stop busy-polling the queue, please signal me when there's data in it
  - how does it know that the other end received the signal?
* please stop polling the queue
* the queue should be deleted

The recipient will always keep a copy of the queue, and will get
signalled when there's data in it.

Which means that all communication between threads is done via
kqueues.  That makes thread discovery / interaction a lot easier.
"Get me kqueue for X".

And kqueues are thread-safe which is nice.  The downside is that if we
want inter-process signalling, we'll need a different signalling
mechanism.

