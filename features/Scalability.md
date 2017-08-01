Scalability
===========

The default configuration that is shipped with the server should work
for most sites under normal load. For high-load systems, there are a few
configuration items should be changed in `raddb/radiusd.conf`:

:   ``` {.config}
          max_requests = 1024
    ```

Change the number `1024` to a larger value, as suggested in the comments
just above that entry.

For systems with a high load and many CPUs, the following configuration
item should also be changed to a higher value:

:   ``` {.config}
          max_servers = 32
    ```

This number should be increased. The best value to use depends on local
load and CPU power. Some sites set this number as high as `500`.

That's it.
----------

The *same* RADIUS server will easily transition from handling one
request every few seconds, to handling thousands of requests per second.
The only changes required to enable this scaling are outlined above.
There is no need to purchase a "enterprise" product license to obtain
better performance, more realms, more RADIUS clients, or any other
feature.

In short, where commercial servers have "small business", "enterprise",
and "carrier class" versions of their product, there is only one version
of FreeRADIUS:

*Carrier class stability and functionality for everyone.*
We prefer to keep it [simple](simple.html).

As proof of it's scalability, the server is currently deployed in
multiple Fortune 500 companies, and Tier 1 ISPs and
telecommunications companies, as seen in the testimonials and
survey pages. Many organizations with more than 10 million
customers are dependant on FreeRADIUS for their AAA needs. For
some of these large sites, and for most small sites (10 to 1000
users) it is the *only* RADIUS server that is used.
