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