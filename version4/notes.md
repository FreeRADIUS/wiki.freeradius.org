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
NAT port, and will therefore be a different connection.