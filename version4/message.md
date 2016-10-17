# Message API

The message API is designed to exchange messages between network and
worker threads.

The v3 design is just a `REQUEST` structure, and a FIFO, along with
mutexes for thread safety.  This design works, but has issues.  For
one, mutex contention is a problem.  Two, memory is often allocated in
one thread and freed in another.  The API is simple, but not
sophisticated.

The goals for the v4 message API are:

* allow it to handle non-RADIUS traffic,
* minimize memory copying
* minimize memory allocation / free
* avoid mutex locks where possible
* be high performance

## Overview

The design of the message API in v4 relies on three components.

First, there is a thread-safe queue of messages.  The queue itself
does not contain the messages, but instead contains pointers to the
messages.  This choice means that the queue is small, and easily
manageable.  It also means that writes to the queue are atomic.

Second, there is a slab allocator message headers.  The message
headers are fixed size, and do not need to be allocated / freed for
every message.

Third, there is a ring buffer for packet data.  This buffer is large
(64K to 1M+), and contains the raw packet data.

An important part of the message implmentation is that there are
multiple instances of the message API (i.e. the three items noted
above).  For example, a network thread may have one instance for each
socket that it's reading from, and a worker thread may have one for
each network thread it's writing packets to.

The reason for multiple instances is not just thread-safety, but
management.  One of the most common problems in asynchronous systems
is data ownership.  Owning data is cheap, exchanging ownership of data
is expensive.

In this model, the system that creates messages is the system that
owns the message.  The message target can (temporily) access the
message, but it does not own the message.

The messages are also ephemeral, and short lived.  If the data is
needed for any length of time, it must be copied out of the message
subsystem to somewhere else.

### Queue

The queue of messages is a simple FIFO set of pointers.  Where
possible, this is done via a lockless queue.  Otherwise, mutexes are
used.

Queues are owned by the recipient of the messages.  i.e. multiple
originators send messages to one queue.  Each recipient (worker
thread, network socket, etc.) has it's own queue.

Messages are placed onto the queue by the originator, and the
destination is signalled that a message is ready.

If the queue has at least one message in it, no signaling is done.  If
the queue transitions from zero messages to N messages, the originator
will signal the recipient, via a queue-specific `kqueue` signal
(EVFILT_USER).

When the recipient receives the message, it will either drain the
queue, or it will have some other way (e.g. socket write ready) of
signalling to itself that it should write more messages from the queue
into the socket.

Similar arguments apply for worker threads.  While they're not writing
to sockets, they can completely drain the queue every time they get a
signal.

The downside of this approach is that there are 6 system calls for
every packet.  One to read the packet, one to signal the queue, one to
receive the signal from the queue, and then the same (in reverse) to
write the reply.  **Lowering this overhead is of high priority.**

### Message Headers

The message headers are allocated via a slab allocator, possibly a
ring buffer.  The message headers are fixed size.

The message API allows for the allocation of a message, possibly with
bulk data.  This allocation just grabs the next free header.  The bulk
data allocation is discussed below.

Once a message has been created and filled out, it is written to the
queue (i.e. a pointer to the message header).  When the destination is
finished with the message, it asynchronously notifies the originator
by marking the message as handled.  The originator is then responsible
for updating the slab / ring allocator with the information that the
message is freed.

This method ensures that there is only a single owner of a message at
a time, and that the originator is the system which does all of the
work for managing memory, lists, etc.

The messages consist of (usually)

* message status (free / allocated / handled)
* message type (send packet, other signal)
* type-specific information
* packet type
  * packet ID (globally unique number)
  * socket information (likely a pointer to the socket information, with src/dst ip/port, file descriptor, etc.)
  * ring buffer pointer
  * pointer / size to raw packet data (if required)
* signal
  * packet ID
  * requested status (done, dup, socket went away, etc.)
* other ...

When the receiver is done with the message, it marks it as "handled",
and lets the originator do asynchronous cleanups.

If the receiver has to sit on the data for a long time
(e.g. `cleanup_delay`), it has to copy the message and any associated
packet data into a separate buffer.  This copy ensures that the
message headers and packet data ring buffer contain information for as
short a period of time as possible.

### Packet Data Ring Buffer

The packet data ring buffer is a way to avoid memory copies, and a way
to deal with variable-sized packets.

The ring buffer starts off at some size (.e.g. 64K), and tries to read
the maximum sized packet into it.  It generally receives a smaller
packet (e.g. 200 bytes), in which case the "start" pointer can be
incremented, and the next packet read into the ring.

Since the "create message" API is used only by one caller
(e.g. network thread or worker thread), there is no issue with
overlapping / simultaneous calls to allocate messages.

## API

The APIs here are documented in reverse order.

### Ring Buffer API

The ring buffers are only used by the message layer, and aren't
directly accessible by the message creators.

    typedef struct fr_ring_buffer_t {
        uint8_t               *buffer;
        size_t                size;

        size_t                reader;
        size_t                writer;

        # Maybe don't need these?
        size_t                num_active_messages;
        fr_ring_buffer_t      *next;
        bool                  should_be_freed;
    }  fr_ring_buffer_t;

Each message API has one ring buffer associated with it, as above.
The buffer has a fixed size.  The `reader` offset is where messages
are read from.  The `writer` offset is where messages are written to.

When a message is freed, the `reader` pointer is incremented.  If
`reader == writer`, then the ring buffer is empty, and both are reset
to zero.

If the writer gets too close to the end (i.e. `writer + max >= size`),
then `writer` is reset to zero.

If `reader > write && (writer + max >= reader)`, then there isn't
enough room to allocate a maximum sized packet in the ring buffer.  A
new ring buffer is allocated, twice the size of the current one.  It
is made as the default ring buffer for the messages.

The old ring buffer is kept around until `reader == writer`, and then
it is discarded.

The message API keeps track of the current and old ring buffers.

Packet allocations from the ring buffer are rounded up to the nearest
cache line size (64 bytes).  This prevents false sharing.

* create - create a ring buffer
* destroy - free a ring buffer (maybe talloc?)
* write - alloc room for N bytes from the ring buffer
* read - read N bytes from the start of the ring buffer

Note that the read / write is done on raw sizes (e.g. 11 bytes), but
the updates to the ring buffer are done internally on cache aligned
sizes (64 bytes, for an 11-byte read/write).

Note that the ring buffer doesn't keep track of where the packet start
/ end is.  It trusts the caller to track that information.

Similarly, the ring buffer API doesn't track previous / next ring
buffers, it relies on the caller to do that.

### Message API

The message API is about allocating a message, and filling it out.

### Queue API

The Queue API is about inserting / removing elements from a FIFO
queue.  The `insert` function returns a special flag if the queue was
empty, so that the originator can poke the receiver that another message is ready?

Or, the even loop / FD has to be exposed to the queue API, so that the
queue code can do this signalling itself.


