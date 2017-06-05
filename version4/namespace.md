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

The `proto_radius` module verifies that all of the packets named here are appropriate, and loads all of the sub-modules.