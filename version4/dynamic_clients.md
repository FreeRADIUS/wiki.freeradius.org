#  Dynamic clients

For scalability and configurability, we need dynamic clients in v4.

Right now, we have static / global clients.  This is fine for 90% of the deployments.

Dynamic clients will let us have clients per listener, per source IP,
per connection (e.g. NAT).

In Phase 1, we will add dynamic clients to the network side, as
dynamic / global clients.  No "per connection" clients will be
created.

In Phase 2, we will add per-connection dynamic clients.

## Phase 1

The network side `proto_radius_udp` needs to be updated to have a
local set of clients.

These clients need "use" counters, so we know how many outstanding
requests are using them.  The clients can only be deleted when the
user counter hits zero.

The dynamic client code needs to be configurable, so that it can be
enabled or disabled.

The dynamic client code needs to have source network ranges where it
accepts packets from.  This is a basic application-layer firewall...

The dynamic clients need a separate virtual server to run for received
packets.  This virtual server looks at the decoded RADIUS packet, and
returns FreeRADIUS VSAs as attributes in the reply.  These reply
attributes are used to create a dynamic client.  The dynamic client is
then inserted into the local tree, with a lifetime.

When `proto_radius_udp` receives a packet from a previously unknown
source, it checks for dynamic clients enable.  If not enabled, it's
rejected.  If enabled, it compares the source IP to the list of
allowed source networks.  If there's no match, it's rejected.
Otherwise, the dynamic client code is started.

For now, we don't check for NAT.  i.e. each client is distinguished
only by source IP, not by port.  If we later add NAT capability,
clients will be distinguished by *connection*, not by source port.

The packet is added *immediately* as a dynamic client, but is marked
with a "pending" flag.  Packets received from this client are *not*
processed, but instead added to a pending queue.  This means having a
local copy of the packets, probably via `talloc()` and memory copies.

The number of these packets should be limited.  That limit should be
configurable.

There should be a timer on these packets.  If the client is not
defined within 1/10s, all of the packets should be discarded.  This
timer should be configurable.

The first received packet is processed through the worker (exactly how
will be described later).

If a NAK is received from the worker, the pending client definition is
removed, and all pending packets are discarded.

If an ACK is received, the format is raw data (not VPs).  The worker
will need to create a client definition, and send it to the network
side as a reply packet.  The network side will then use those fields
to update the "pending" client definition, and remove the "pending"
flag.

At that point, the pending packets are removed from the queue, and
processed through the normal network side.

*TODO* How to do that?  The `write()` function doesn't have access to
 anything except it's local socket data and the raw packet.  It can't
 poke the network...

We can't rely on the `read()` routine being called, as we might not
receive new packets for a while.  So we need some way to trigger the
`read()` from the `write()` function.

Maybe just change the `write()` API to return `WANT_READ`, ala
OpenSSL.  That's by far and away the simplest approach.

### Corner cases

This code needs to use the standard packet tracking mechanisms.
i.e. if a duplicate is received, ignore it.  If a conflicting packet
is received, drop either the new one or the old one.

But we don't want to pollute the main table with packets / IPs for
unknown clients, so we use a special "pending" tracking table.  Then
when pending packets are moved over, copy re-insert them into the main
table.

much of the bottom half of `proto_radius_udp`, function `mod_read()`
can be turned into another function.  The "undo pending" then just
pulls the pending packets off of a queue, and calls the bottom-half
function.

### Changes

* new tracking table for pending entries
* `fr_dlist_t` in inst, for pending packets
* "active" flag in clients
* `fr_dlist_t` in clients of pending packets
* new data structure for pending packets which holds:
  * ptr to tracking table entry
  * ptr to packet data
  * `fr_dlist_t` for client entries
  * ptr to client

## How things will work

### proto_radius_udp, mod_read:

* if inst->list (i.e. there are pending packets)
  * remove entry from list
  * copy address, client, packet
  * return `mod_read_p2()`

* otherwise read from socket, etc.

* look in global list.  If not found, look in dynamic client list
  * if !found && in allowed networks, return dynamic_client_alloc(inst, packet, length, address)
    * otherwise disallow, and return 0
    * this is a negative cache entry.  Update the expiry time, by adding 30s.
  * if found, && !client->active, return `dynamic_client_check(inst, packet, length, address, client)`
    * otherwise it's an active dynamic client, it's OK

* otherwise return `mod_read_p2(...)`

### Function `dynamic_client_alloc()`

* if inst->num_pending_clients is too many, return 0

* increment inst->num_pending_clients

* alloc the client, parented from the instance
* fill in various fields (e.g. src IP)
* set `client->active = false`
* insert client into dynamic client list
* call `new_struct_alloc()`
* insert it into `client->list`
* return the packet to the network code


### Function `new_struct_alloc()`

* allocate new structure, parented from dynamic client and fill it in
  * tracking table entry,
  * dlist
  * ptr to client
  * talloc_memdup() the packet

 * add to tail of `client->list`


### Function `dynamic_client_check()`

* if client->num_pending_packets is too many, drop
* look up packet in inst->dynamic_client_tracking
  * do similar logic as in bottom half of `mod_read()`
  * but skip the packet verify, stats, etc.
  * SAME -> drop packet
  * UPDATED - shouldn't happen (this is new packet after old reply)
  * CONFLICTING - drop packet
  * other - drop packet
  * NEW, continue

* call `new_struct_alloc()`
* increment client->num_pending_packets

* return 0
 * we rely on other things to clean up the client && it's packets

### function `mod_set_process()` in proto_radius

* if !client->active
  * set `request->process` to `proto_radius_dynamic_client()`
  * set methods to 'new client'

### function `mod_encode()`

* If !client->active
  * look for attributes to define packet, and create client definition if so
  * encode it as raw data, or encode 1-byte "nak" if not found

### function `mod_write()`

* if !client->active
  * decode raw packet as either client or NAK
  * if NAK, free all of the pending packets
    * set `client->unknown`, and set cleanup timer,
    * return
  * if OK, update client definition
    * set client to active
    * decrement inst->num_pending_clients
    * move list of pending packets to inst->list
    * return WANT_READ

*Note* we have to fix all mod_write() to return a status `int` instead of number of bytes written
* <0 = error
* 0 = close the socket
* 1 = wrote data OK
* 2 = WANT_READ

### Function in network.c calling `write()`

* if `write()` returns WANT_READ
  * call `fr_network_read()` to empty the pending queue for this socket