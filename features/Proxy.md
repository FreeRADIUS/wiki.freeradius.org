Proxying
========

The server can [proxy](http://wiki.freeradius.org/Proxy) any request to
other RADIUS servers. Proxying can be done via the common RADIUS
`realms` method, which is completely under the control of the local
administrator. Both prefixes and suffixes are support, as are other
arbitrary methods of selecting a realm.

[Policies](Policy) can be executed before, and after proxying. This
capability enables the administrator to re-write requests before they
are proxied, and also to re-write replies from home servers. For
example, IP addresses may be assigned [locally](IPPool), so any IP
address assignment received from a home server needs to be deleted.

Home servers are grouped into pools, which permit multiple kinds of
fail-over and load-balancing:

-   Simple fail-over. The first home server is used, unless it is down.
    In which case the second is used, and so on.
-   Load-balancing. Requests are split evenly among all home servers in
    the pool.
-   Load-balancing by client. Requests from a client are automatically
    mapped to one home server by hashing the client IP address. There
    can be a different number of clients and home servers, in which case
    many clients may map to one home server.
-   Load balancing by client port. As above, except that the source port
    is included in the hash calculation.
-   Keyed hashing. Similar to the two previous methods, but the hash key
    can be created dynamically by the administrator. This configuration
    permits, for example, EAP requests to be sent to one home server,
    and all other authentication requests be sent to another.
