# The IO Layer

The IO layer is responsible for base IO protocols.  i.e. bare-bones
IO.  All memory is talloc'd, and functions which require memory
allocation take a `TALLOC_CTX *`.  Freeing is done via talloc.

Additional notes on UDP are in the [UDP](udp) page.

## Thread Safety

The IO layer is not thread-safe.  That is, it is the callers
responsbility to ensure that only one thread at a time uses a
particular `fr_io_t`.

Where we have parent-child relationships, we could (if we cared)
create message queues between them for signalling.  That would allow
different threads to handle parent and child sockets.  Which might be
useful...

Otherwise, only one thread can handle parent / child sockets.

The IO layer is used by the transport layer to connect new sockets.
The transport layer then needs to notify the [thread](threads) API
(somehow) that there is a new socket.

### parse

Takes a `CONF_SECTION` and returns a `fr_io_t *`.  It does all
of the work of parsing type-specific information.

Maybe we also need a `parse_client` and `parse_server`, especially for
TCP, TLS, etc. where the behaviors are rather different.

We probably want to separate TLS parameters from TCP.  i.e. so that
they're independent.  This lets us use TLS + TCP, or TLS + UDP if we
want.

If we have nested parsers, then the debug output will also be nested,
which is perhaps not what we want??  Unless one parser plays games
with the other, so that it's not indented...  which is annoying.

maybe allow "protocol=tls+tcp" ? which at least makes it clear what's
going on.

### print

Takes a `fr_io_t *` and returns a `char const *`, which is
talloc'd.  The string is a short description of the method.

### open

Takes a `fr_io_t *` and returns an `int` (0/-1).  It is a
server-side function that opens a connection, and where necessary,
binds to a socket.

### close

Takes a `fr_io_t *` and returns an `int` (0/-1).  Closes the
connection.

For accepted sockets (i.e. children), also removes the child from the
parent.

### accept

Takes a `fr_io_t *` and returns a `fr_io_t *`.  It is a
server-side function for stream sockets.  For other type of
connections it can be NULL.

The accepted socket is also tracked as a child of the listener socket.

### connect

Takes a `fr_io_t *` and returns a `ssize_t`. Connects an
outgoing socket.  If the socket requires retries to connect, that
state is saved in the `fr_io_t`, and the relevant read/write
function calls connect again until it succeeds.

Errors are returned as (e.g.) `-EAGAIN`

### read

Takes a `fr_io_t *`, `char const *data`, and `size_t data_len`
and returns a `ssize_t` of data read, or `-ERROR`.

### write

Takes a `fr_io_t *`, `char const *data`, and `size_t data_len`
and returns a `ssize_t` of data written, or `-ERROR`.

### event

Takes a `fr_io_t *`, `fr_event_list_t`, along with read/write
callbacks.  Inserts itself into the event loop, and sets up the callbacks for read/write readiness.

The read/write callbacks should be supplied by the caller, and will
call the `read` and `write` routines to write to buffers.

If `read` or `write` are NULL and exist in the event loop, the events
are removed.  This lets the callback move between event loops.

i.e. *all memory management is done by the caller*.

### debug

Takes a `fr_io_t *` and ???, and prints debugging information about the method.

