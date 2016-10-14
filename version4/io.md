# The IO Layer

The IO layer is responsible for base IO protocols.
i.e. bare-bones IO.  All memory is talloc'd, and functions which
require memory allocation take a `TALLOC_CTX *`.  Freeing is done via
talloc.

Additional notes on UDP are in the [UDP](udp) page.

## Thread Safety

The IO layer is not thread-safe.  That is, it is the callers
responsbility to ensure that only one thread at a time uses the API.

### parse

Takes a `CONF_SECTION` and returns a `fr_io_t *`.  It does all
of the work of parsing type-specific information.

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

### accept

Takes a `fr_io_t *` and returns a `fr_io_t *`.  It is a
server-side function for stream sockets.  For other type of
connections it can be NULL.

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

