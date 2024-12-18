# Network stats

The network side will keep it's own statistics, e.g. `rlm_radius_udp.c`.  Except for a few RADIUS-specific things, these are largely protocol-independent:

    typedef struct fr_stats_t {
        fr_uint_t	total_requests;
        fr_uint_t	total_responses;
        fr_uint_t	total_invalid_requests;
        fr_uint_t	total_dup_requests;
        fr_uint_t	total_malformed_requests;
        fr_uint_t	total_bad_authenticators;
        fr_uint_t	total_packets_dropped;
        fr_uint_t	total_unknown_types;
    } fr_stats_t;

i.e.

* total input packets
* total output packets
* "invalid", i.e. ones from unknown sources
* duplicates
* malformed
* fail a signature check
* dropped for other reasons
* packets we don't allow on this listener.

Any protocol-specific statistics (e.g. Access-Accept vs Access-Challenge) are in a separate protocol-specific data structure, also in the `proto_radius` code.

The API to get these statistics can just be a method in `app_io`.  The worker can grab these stats directly, even if the memory is on the network side.  Since the statistics are being incremented only, just doing a `memcpy()` of the structure should be OK.  Any thread-safety issues can be ignored, as on x86-64, 64-bit writes are atomic.

i.e. the stats API should

* be called from the work to the network side
  * because the request is still alive, we know that the network side is still alive
* `memcpy()` the stats from the network side instance to a thread-local variable on the stack
  * not thread-safe, but who care.  If the stats are off by 1 or 2 it doesn't matter
* build up stats as TLVs and add them to the request

## Stats module

Ideally, we allow the admin to track statistics by any metric.  Client IP, listener IP/port, home server IP/port, etc.

For home server, this *may* mean adding stats capability to the RADIUS client module.  My original idea was to have `send Access-Request` and `recv Access-Accept` sections, in which we could do things like list the `stats` module.

The current design has the RADIUS module just send packets directly, and adds *all* of the reply attributes to `request->reply`.  Along with that, the reply packet code is unavailable, too.  This says to me that we should really make the `radius` module have it's own sub-sections that it runs, which gives the admin  greater control over the proxied requests and responses.  That would also then allow home server stats to be trivially added: list `stats` in the appropriate `send` or `recv` section.

### Configuration of the stats module

Ideally, we wouldn't need one `stats` module per statistic we want to keep.  Instead, it should be possible to keep multiple statistics. e.g. by client IP, listener IP, etc.

The `stats module needs to update multiple statistics in one pass.  Ideally:

* stats about the source of the packet (i.e. client)
* stats about the destination of the packet (i.e. listener)
* global server statistics
* *maybe also virtual server statistics*

TBH, the simplest thing may be for the module to have an rbtree to track the stats.  It looks up the thing (source / dest / virtual server), finds the stats, and increments them.  The stats can be queried then by "SRC + key", or "DST + key", or "SERVER + key", or "global"

Since all of the src/dst data is (or should be) available in the request, this shouldn't be too hard to do.  We don't need xlats or string parsing.  We can just look at `request->packet->src_ipaddr` etc. directly.

For destination, we need to look at `dst IP + dst port`

For dynamic clients, we need to look at `NAS-IP-Address` or `NAS-Identifier`, *instead* of `src IP`.  That's likely an optional additional key for the stats module.

It's probably best to have protocol-specific stats modules, as different statistics will need different things.

### Querying the stats

via magic.  `virtual_servers.c` should have a way to call a method across all threads of a module:

    coalesce(module_global_data, module_thread_instance)

and then to pass the results back to a particular thread:

    finish(module_global_data, module_thread_instance)

That way a module can request that all of the stats be merged from all threads.

We will ignore thread-safety again here... locks are too expensive.  i.e. the stats will be updated millions of times per second, but read only once per second.  The reader doesn't care if the stats are off by 1, as there will always be "in flight" packets.  So this approach is likely correct.

We would likely need protocol-specific modules... so that the `stats` module could be listed in a `recv Status-Server` section.  It would then look for queries, coalesce the stats, and update the reply as appropriate.  This would involve largely taking the v3 `stats.c` code, and moving it into a module.

### Historical data

The server should only track "live" data.  If someone wants historical data, they should query the server periodically.

It also means that the module should not clean up stats for dynamic clients / home servers.  Due to data abstraction, the module *cannot* know if a client has gone away.  All it knows is that the client is no longer sending packets.

This also means that there should be a way to delete stats for a particular thing (client, listener, etc.)  So the action of a client expiring can (if the admin desires) result in the stats being deleted.  Or possibly better, the stats are cleared when a dynamic client is created.

### SNMP and Statistics

SNMP requires an ID to index logical tables of statistics.  This ID has to be contiguous, and stable.  I'm not sure how to do this.  Especially in the context of dynamic clients, where a client can come and go.

In v3, it just assigns a random number, and tells the caller to check the table contents (e.g. IP address, client ID) for client identity.  This is probably the thing to do here.

### Normalization

We should produce stats in one form, and then rely on "unlang" to get them to different attributes as necessary.

e.g. old-style stats, or SNMP stats.

