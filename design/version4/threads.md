# Threading

Version 4 should be async by default.  There should be minimal
communication between threads.  All necessary communication should be
through [messages](message).

There are network threads, which read / write to the network.  There
are worker threads, which process packets (unlang, SQL, etc).

We probbly want thread affinity, where a network thread preferentially
sends packets to one worker thread.  In the common case, a worker
thread processes packets faster than they arrive.  So distributing
packets among worker threads isn't necessary.

In the high performance case, there may be bursts where offered load
is higher than accepted load.  The packets should then probably wbe
distributed among worker threads.

We would also like affinity for EAP authentications.  i.e. All packets
for an EAP-TLS conversastion should go to one worker thread.  Doing so
means that we can get rid of most OpenSSL mutexes, as each `SSL_CTX`
is thread-local.

## Notes on UDP writes

In many cases, worker threads can write replies directly to the UDP
socket.  This is especially true for accounting packets, where there
is no de-dup and no cleanup delay.  Where there is no de-dup or
cleanup delay, the [transport](transport) front-end can receive a
packet, send it to the worker thread, and then forget all about the
packet.

Things get more difficult for authentication packets, which are
required to de-dup and have cleanup delay.  We would like to avoid
mutexes where possible, so the question is which is cheaper?  Sending
the reply from the worker to network thread, and doing de-dup updates
/ cleanup delay there?  Or having network + worker thread both access
a mutex-protected dedup tree?  The issue with mutexes is that every
*access* to the tree has to be mutex protected, which is very bad...

## Notes on TCP Writes

The current proposal is to use [messages](message) to exchange
packets.  The worker thread builds the raw packet, and sends it to the
network thread.  In the general case, the network thread writes the
packet to the socket and it's done.  If the socket isn't ready, (or
auth de-dup / cleanup delay), the packet has to be copied out of the
message buffer into a local buffer for later writes.  At that point,
the network thread can just try to write all available data, and can
ignore packet boundaries.

This kind of design can cause issues, though. See:
https://www.socallinuxexpo.org/sites/default/files/presentations/2014SCALE-hyc.pdf

A different design would be to get rid of the writer thread entirely.
In this model, when a worker thread needs to write to a TCP
connection, it grabs a "write context" for that TCP connection.  This
means that it is the thread which is writing to the TCP connection.

The main utility here is for slow clients.  Instead of having a writer
thread which is mostly idle, you can have worker threads which do
other things.

i.e. if the socket is writable (which is the general case), the worker
thread just writes to the socket.  (with caveats for packet
boundaries, threading issues, etc).  If the socket isn't writable, the
worker thread can add it's raw packet to the outbound queue.

When the socket becomes writable, the worker thread writes all pending
packets to the socket, and then goes on with other business.  Again,
this is mainly useful for slow connections.  For fast connections, the
worker thread can probably just grab a mutex, write the data, and be
done.

This design means that the worker threads would not call the message
API to send a message to the network threads inbound message
queue. Instead, they would call a transport API (which is almost
identical to the message API) to send the message.
e.g. `transport_msg(transport, ....)`.  This API means that the
transport can choose to send the message to the network thread via the
message API, or (if allowed by the transport), it camn just send the
message and avoid the overhead of messaging.

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
But it may be useful to do. See [UDP](udp) for more notes on RSS.

We also have the issue that in the common case (probably 90% of
deployments), the incoming packet rate is very low.  i.e. a packet
comes in, gets processed, and the server becomes idle until another
packet comes in.  We want to have a simple way to deal with that
use-case.  Having multiple threads and lots of coordination is
overkill, and will work, but it's not overly efficient.

We may be able to use "self clocking" for handling business.
i.e. each packet received / sent has a timestamp.  So, we use that as
a rough guide to how much time the server spent processing the packet.
