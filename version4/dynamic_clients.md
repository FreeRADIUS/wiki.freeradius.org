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

1.1 The network side `proto_radius_udp` has a
local set of clients.

1.2. These clients have an "outsranding" counters, so we know how many outstanding
requests are using them.  *Even when expired, the clients can only be deleted when this counter hits zero.*

1.3. The dynamic client code is configurable, so that it can be
enabled or disabled.

    dynamic_clients {
        ...
    }

1.4. The dynamic client code has source network ranges where it
accepts packets from. 

    dynamic_clients {
        network = 192.0.2.0/24
        network = 127.0.0.0/8
        ...
    }

1.5. The dynamic clients run a `new client` subsection of the current virtual server, for received
packets.  This subsection looks at the decoded RADIUS packet, and
returns FreeRADIUS VSAs as attributes in the reply.  *These reply
attributes are used to create a dynamic client.  The dynamic client is
then inserted into the local tree, with a lifetime.*

1.6. When `proto_radius_udp` receives a packet from a previously unknown
source, it checks for dynamic clients enabled.  If not enabled, it's
rejected.  If enabled, it compares the source IP to the list of
allowed source networks.  If there's no match, it's rejected.
Otherwise, the dynamic client code is started.

For now, we don't check for NAT.  i.e. each client is distinguished
only by source IP, not by port.  If we later add NAT capability,
clients will be distinguished by *connection*, not by source port.

1.7. The packet is added *immediately* as a dynamic client, but is marked
with a "pending" flag.  Packets received from this client are *not*
processed, but instead added to a pending queue.  This means having a
local copy of the packets, probably via `talloc()` and memory copies.

1.8. The number of these packets  is limited.  That limit is
configurable.

    dynamic_clients {
        max_pending_packets = 65536
        ...
    }

1.9. *There should be a timer on these packets.  If the client is not
defined within 1/10s, all of the additional packets should be discarded.  This
timer should be configurable.*

1.10. The first received packet is processed through the worker (exactly how
will be described later).*

1.11. If a NAK is received from the worker, the pending client definition is
removed, and all pending packets are discarded.

1.12. *If an ACK is received, the format is raw data (not VPs).  The worker
will need to create a client definition, and send it to the network
side as a reply packet.  The network side will then use those fields
to update the "pending" client definition, and remove the "pending"
flag.*

1.13. At that point, the pending packets are removed from the queue, and
processed through the normal network side, by calling `fr_network_listen_read()`

### Corner cases

This code uses the standard packet tracking mechanisms.
i.e. if a duplicate is received, ignore it.  If a conflicting packet
is received, drop either the new one or the old one.

## How things will work

### proto_radius_udp, mod_read:

* if inst->list (i.e. there are pending packets)
  * remove entry from list
  * copy address, client, packet
  * do bottom half of processing from `mod_read()`

* otherwise read from socket

* look in global list.  If not found, look in dynamic client list
  * if !found && in allowed networks, return dynamic_client_alloc(inst, packet, length, address)
    * otherwise disallow, and return 0
    * this is a negative cache entry.  Update the expiry time, by adding 30s.
  * if found, && !client->active, return `dynamic_client_check(inst, packet, length, address, client)`
    * otherwise it's an active dynamic client, it's OK

* otherwise do bottom half of processing from `mod_read()`

### Function `dynamic_client_alloc()`

* if inst->num_pending_clients is too many, return 0

* increment inst->num_pending_clients

* alloc the client, parented from the instance
* fill in various fields (e.g. src IP)
* set `client->active = false`
* set `client->dynamic = true`
* insert client into dynamic client list
* call `dynamic_client_packet_save()` to save the packet
* return the packet to the network code

### Function `dynamic_client_packet_save()`

* if client->num_pending_packets is too many, drop
* look up packet in inst->dynamic_client_tracking
  * do similar logic as in bottom half of `mod_read()`
  * but skip the packet verify, stats, etc.
  * SAME -> drop packet
  * UPDATED - shouldn't happen (this is new packet after old reply)
  * CONFLICTING - drop packet
  * other - drop packet
  * NEW, continue

* allocate new structure, parented from instance and fill it in
  * tracking table entry,
  * dlist
  * ptr to client
  * talloc_memdup() the packet

 * add to tail of `client->list`

* increment client->num_pending_packets

* return 0
 * we rely on other things to clean up the client && it's packets

### function `mod_set_process()` in proto_radius

* if !client->active
  * set `request->process` to `proto_radius_dynamic_client()`

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
  * *if OK, update client definition*
    * set client to active
    * decrement inst->num_pending_clients
    * move list of pending packets to inst->list
    * call `fr_network_listen_read()`

### proto_radius_dynamic_client()

Has methods:

* `new client` - 'recv' section
* `add client`  - 'send' section
* `deny client` - 'send reject' section

It also looks at the attributes necessary to define a client.  If they are missing or incomplete, it complains, and runs the 'deny client' section.