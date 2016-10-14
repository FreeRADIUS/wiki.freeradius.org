# Threading

Version 4 should be async by default.  There should be minimal
communication between threads.  All necessary communication should be
through thread-safe queues.

We need a threading API which is separate from everything else.  It
takes care of queueing requests, signalling them, etc.

### create

Takes `CONF_SECTION *` and returns `fr_worker_thread_t *`.

### kill

Takes `fr_worker_thread_t *`, and ???, and signals the thread to exit.
The signals can be NOW, or terminate gracefully.

### enqueue

Takes `fr_thread_t *` and `REQUEST *`, and enqueues the request for the thread to process.

*Maybe we just need one enqueue function, which enqueues a request and a signal?*

And the default is `FR_ACTION_QUEUE` ?

### signal

Takes `fr_thread_t *` and `REQUEST *`, and `fr_action_t *`, and signals the request.

*We need this to be async safe, so we may need a REQUEST from the
network thread, which is then duplicated as a child REQUEST in the
worker thread?*`

## Notes on TCP Writes

The usual way to handle TCP writes for multiple worker threads
(e.g. radsecproxy), is to have (a) a thread each to read/write the
socket, and (b) worker threads.  Each thread has a queue associated
with it.  Packets which are read go to a queue on a worker thread.
When the packets are read to be written, the worker thread places them
onto the queue for the writer thread.

This can cause issues:  https://www.socallinuxexpo.org/sites/default/files/presentations/2014SCALE-hyc.pdf

A different design is to get rid of the writer thread entirely.  In
this model, when a worker thread needs to write to a TCP connection,
it grabs a "write context" for that TCP connection.  This means that
it is the thread which is writing to the TCP connection.

The main utility here is for slow clients.  Instead of having a writer
thread which is mostly idle, you can have worker threads which do
other things.

When the socket becomes writable, the worker thread writes all pending
packets to the socket, and then goes on with other business.  Again,
this is mainly useful for slow connections.  For fast connections, the
worker thread can probably just grab a mutex, write the data, and be
done.

## Notes on Performance

Not withstanding the OpenLDAP experience, we may want to do something
different.

Analysis of v2 / v3 shows that very often the bulk of the CPU time is
spent in the listener thread (of which there is one).  i.e. under high
load, a huge portion of the time is spent reading the sockets, doing
de-dup, and very little is spent processing the packets.

We may want to fix this in v4. :)

One way (for UDP sockets) is to note that we can have multiple readers
on the same socket.  There is no way around the "thundering herd"
problem, where if one packet is available... all sockets get woken up.
But it may be useful to do.

We also have the issue that in the common case (probably 90% of
deployments), the incoming packet rate is very low.  i.e. a packet
comes in, gets processed, and the server becomes idle until another
packet comes in.  We want to have a simple way to deal with that
use-case.  Having multiple threads and lots of coordination is
overkill, and will work, but it's not overly efficient.

We may be able to use "self clocking" for handling business.
i.e. each packet received / sent has a timestamp.  So, we use that as
a rough guide to how much time the server spent processing the packet.
