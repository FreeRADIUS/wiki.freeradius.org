# Proxy configuration for version 4

Proxying has changed vastly for version 4. The `proxy.conf` file
no longer exists, and all proxying is now done in a new RADIUS
client module, `rlm_radius`. This moves proxying out of the server
core.

The downside to this is that configuring the server for proxying
is now more verbose. The upside is the flexibility gained by
having a full RADIUS client available to call from unlang wherever
it is needed.


## Version 3 example

As a starting point, a version 3 proxy configuration looks
something like the following.

`proxy.conf` defines individual servers to proxy to, for example:

    home_server remoteradius1 {
        type = auth
        ipaddr = radius1.network.example
        port = 1812
        secret = aS3cr3T
    }

It also defines server pools, which are used for redundancy and
failover between remote RADIUS servers e.g.

    home_server_pool remote_radius_failover {
        type = fail-over
        home_server = remoteradius1
        home_server = remoteradius2
        home_server = ...
    }

Finally, it contains realms, which are essentially just labels
defining which home server or pools to use for proxying:

    realm network.example {
        auth_pool = remote_radius_failover
    }

Proxying happens after the `authorize {}` section in a virtual
server configuration. If the `Proxy-to-Realm` control attribute is
set, the server looks for a realm of the same name in the
`proxy.conf` file. For example:

    authorize {
        preprocess
        files
        update control {
            Proxy-to-Realm := "network.example"
        }
        pap
    }

All incoming requests here will be proxied. When the server gets
to the end of `authorize`, it will see that `Proxy-to-Realm` is
set to "network.example". It will then skip authentication
entirely and instead look up `realm network.example` in
`proxy.conf`. From this it will find out that it needs to use pool
`remote_radius_failover`. This will tell it the home servers to
use, and the request will be sent off to one of them according to
internal load-balancing or failover rules.

There is no actual requirement that the realm names in proxy.conf
are the same as the realms in the User-Name attribute of the
incoming request, they are purely labels. However it can often be
convenient to keep them the same. This is where the `rlm_realm`
module comes in. It takes the incoming User-Name and extracts the
realm from it, putting it into the `Proxy-to-Realm` attribute.
`suffix` is defined in the default configuration as an
instantiation of `rlm_realm` that strips the realm after the @ in
the User-Name.

However, it does not matter how `Proxy-to-Realm` is set - by
unlang or any other module - only that the server core uses this
to choose the realm to proxy to.


## Changes in version 4

Version 4 is completely different in practice, though the concept
is the same. The `Proxy-to-Realm` attribute has gone, as there is
no longer any special hook in the server core to intercept the
request before authentication and proxy instead.  Therefore the
`proxy.conf` file has also gone - proxy servers are now defined
simply as instances of the `rlm_radius` module. All hidden
complexity of proxy pools, load-balancing or failover are now in
the open in unlang. Proxying is simply an alternative form of
authentication - it just happens to be remote, rather than on the
local server.

Therefore, rather than defining home servers in `proxy.conf`, home
servers are created by new instances of `rlm_radius`. This may for
example contain the following directives:

    radius remoteradius1 {
        transport = udp
        type = Access-Request
        udp {
            ipaddr = 192.0.2.1
            port = 1812
            secret = aS3cr3T
        }
    }

To proxy an incoming request, we now define a new authentication
type, for example `proxy-network.example` - this will act in the
same way as the `realm` and `home_server_pool` options did
previously in `proxy.conf`:

    authenticate proxy-network.example {
        redundant-load-balance {
            remoteradius1
            remoteradius2
            ...
        }
    }

At this stage, we are now just missing the final component -
telling the server to proxy the request. The configuration is very
similar to before, but instead of setting `Proxy-to-Realm`
control attribute, we now just set Auth-Type:

    recv Access-Request {
        preprocess
        files
        update control {
            Auth-Type := "proxy-network.example"
        }
        pap
    }


## Quick reference

A general summary of configuration required to convert the proxy
configuration from version 3 to version 4:

  * Create an instance of `rlm_radius` for each home server from
    `proxy.conf`.

  * Add a new `authenticate` section for each realm, which calls
    the appropriate home servers.

  * Update the new `recv` section (which replaces `authorize`)
    with unlang to set `Auth-Type` appropriately, rather than
    `Proxy-to-Realm`.


## More complicated setups

In larger configurations there may be a large number of home
server pools, each of which is called from a number of realms.
In version 4, policies can be used to maintain the same level of
indirection.

For example, say there were two home server pools, `pool_a` and
`pool_b`, and twenty realms, ten of which used one pool, and ten
the other. Listing the home servers of `pool_a` in ten realms, and
the servers of `pool_b` in another ten realms, starts to lead to a
lot of duplication.

Instead, create a new policy file, `raddb/policy.d/proxy` and
define two policies, `proxy_pool_a` and `proxy_pool_b`:

    proxy_pool_a {
        redundant-load-balance {
            home_server_1
            home_server_2
            home_server_3
            home_server_4
            home_server_5
        }
    }

    proxy_pool_b {
        redundant-load-balance {
            home_server_6
            home_server_7
            home_server_8
        }
    }

Now, in the authenticate sections for each realm, just call the
appropriate policy:

    authenticate proxy-realm_1.example {
        proxy_pool_a
    }

    authenticate proxy-realm_2.example {
        proxy_pool_b
    }

The policies are now equivalent to the old home server pools in
version 3, and the named authenticate sections are equivalent to
realms.

## Why this change was made

This change was made in order to permit new features which were long requested in previous versions of the server.  Due to design limitations, these features were impossible to implement.

Until now.

Please see the [Proxy Extensions](proxy-extensions) page for more information.