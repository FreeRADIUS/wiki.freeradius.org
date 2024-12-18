# Namespaces in the new virtual servers

Each virtual server section starts off with a namespace:

    server foo {
        namespace = radius
        ...

The code in `virtual_servers.c` sees the `namespace`, and the follows the process outlined below.

The main server core (`radiusd.c`) has already created a `fr_schedule_t`, and passed it to `virtual_servers_init()`.  This scheduler is responsible for all network and worker threads.

## Loading the main protocol

When there is `namespace = radius`, the code tries to load a library `proto_radius`, and link to `fr_app_t proto_radius`.

That structure contains the functions necessary to load the library, and parse the configuration for this namespace.

Note that unlike v3, the `parse` function does *not* return a listener that is managed by the server core.  Instead, the `parse` function takes a `fr_schedule_t` (i.e. scheduler core), and inserts a socket directly into the scheduler.

The `proto_radius` library does all necessary initialization for this protocol / application.  Typically, this effort involves loading sub-modules which implement the processing state machine, and other sub-modules which open network sockets.  The main `proto_radius` library glues these two sub-modules together to allow the scheduler to process packets.

The sub-module configuration is stored in a `listen` section, as in v3:

    listen {
        ...
    }

### State Machine Submodule

The next item which gets parsed is the following:

    listen {
        packet = Status-Server
        ....

The `proto_radius` module converts this configuration into loading a library `proto_radius_status`.  That library contains the state machine for handling Status-Server packets.  i.e. parsing the `recv Status-Server` and `send Access-Accept` sections, and the state machine for processing them.  Note that this library does no IO, and does not encode or decode any packets.

The `proto_radius_status` library may parse it's own configuration options, typically in a sub-section named for the packet type:

    listen {
        packet = Status-Server
        ...
        Status-Server {
            ...
        }
        ...

There may be multiple `packet` configurations:

    listen {
        packet = Status-Server
        packet = Access-Accept
        ...

The `proto_radius` module verifies that all of the packets named here are appropriate, and loads all of the state machine sub-modules.

Once each state machine sub-module has been loaded, the IO submodules are loaded.

### IO Submodules

Each `listen` section also has a `transport` configuration item.  This configuration controls what transport layer is being used for packets.

The main goal of these changes is to enforce a hard separation between the state machines for processing packets, and how those packets are received or sent.  The v3 code did not have such a separation, which meant that proxying and "originate CoA" were one-offs.  They were welded into the server core, and could not be changed without major work.  In addition, we couldn't add new protocols without changing the server core, which meant (in all likelihood) accidentally breaking the existing state machines.

The new configuration looks like this:

    listen {
        packet = Status-Server
        ...
        transport = udp
        ...

Where the transports are typically `udp`, `tcp`, `file`, etc.

Note also that there is only one IO submodule.  While it's likely not difficult to allow for multiple IO sub-modules, it's better to keep the initial design simple.

The `proto_radius` module looks for the `transport` configuration, and loads the appropriate IO sub-module.  In this case, `proto_radius_udp`.

There is typically transport-specific configuration, as follows:

    listen {
        packet = Status-Server
        ...
        transport = udp
        ...
        udp {
            ipaddr = 192.0.2.25
            port = 1812
            ...
        }
        ...

The `proto_radius_udp` module is responsible for parsing the `udp` sub-section.  Once parsed, it returns:

* socket FD
* transport context
* `fr_transport_t` data structure

The `fr_transport_t` data structure contains all of the functions necessary to read/write the socket.  The transport context is transport-specific context, such as de-dup trees, etc.

Note that the IO functions *must* be able to handle reading and writing all possible kinds of packets.  It has no knowledge as to whether or not the source IP is allowed (e.g. known clients).  It has no knowledge of which packets are allowed.  It has no knowledge of shared secrets, etc.

The benefit of this division of responsibility is that the IO layer has a hard separation from the state machine.  The downside is that the IO layer has to do a bit more work, in that it has to handle all possible packet types.  e.g. do de-dup, cleanup delay, and reject delay for Access-Request, but not do that for Status-Server or Accounting-Request.

The extra complexity in the IO layer is worth it, as it is too difficult for now to completely abstract the MxN matrix of packet code vs transport.

Once the `proto_radius_udp` IO module returns the triple of (socket, context, transport), the `proto_radius` main module is responsible for gluing together the state machine and IO pieces.

## Gluing it all together.

The `proto_radius` main module now has all of the information it needs to create a listener.  It does the following, in pseudo-code

    for each listen section
        for each state machine sub-module
            for the one IO sub-module
                allocate new `fr_transport_t`
                copy IO `fr_transport_t` to the newly allocated one
                update the `process` function with the one from the state machine submodule
                allocate new context = (transport context, allowed packets)
                insert (sockfd, context, transport) to the scheduler

Note that the `context` here is one for the `proto_radius` module.  That context tracks which packets are allowed, and the underlying IO context, among other pieces of information.

Once all of this work is done, the `proto_radius` main module returns to the server core, and the next virtual server is parsed.