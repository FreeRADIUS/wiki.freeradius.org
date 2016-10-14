# The Transport Layer

The Transport layer is responsible gluing the IO layer to the
messaging / application layer.  All memory is talloc'd, and functions
which require memory allocation take a `TALLOC_CTX *`.  Freeing is
done via talloc.

## Similarity to IO layer

Many of the APIs are similar to the IO layer.  The transport layer
adds application awareness to the IO layer, but otherwise behaves much
like the IO layer.

## Thread Safety

The transport layer is not thread-safe.  That is, it is the callers
responsbility to ensure that only one thread at a time uses the API.

### parse

Takes a `CONF_SECTION` and returns a `fr_transport_t *`.  It does all
of the work of parsing type-specific information.

### print

Takes a `fr_transport_t *` and returns a `char const *`, which is
talloc'd.  The string is a short description of the transport.

### open

Takes a `fr_transport_t *` and returns an `int` (0/-1).  It is a
server-side function that connects the IO layer to the underlying
application.

### close

Takes a `fr_transport_t *` and returns an `int` (0/-1).  Closes the
connection.

### accept

Takes a `fr_transport_t *` and returns a `fr_transport_t *`.  It is a
server-side function for stream sockets.  For other type of
connections it can be NULL.

### connect

Takes a `fr_transport_t *` and returns a `ssize_t`. Connects an
outgoing socket.  If the socket requires retries to connect, that
state is saved in the `fr_transport_t`, and the relevant read/write
function calls connect again until it succeeds.

Errors are returned as (e.g.) `-EAGAIN`

### read

Takes a `fr_transport_t *`, `RADIUS_PACKET **`
and returns a `ssize_t` of data read, or `-ERROR`.

Note that unlike the IO layer, the transport layer reads application
data.  This is because it needs to be application aware.

### write

Takes a `fr_transport_t *`, `RADIUS_PACKET *`
and returns a `ssize_t` of data written, or `-ERROR`.

Note that unlike the IO layer, the transport layer writes application
data.  This is because it needs to be application aware.

### event

Takes a `fr_transport_t *`, `fr_event_list_t`, along with read/write
callbacks.  Inserts itself into the event loop, and sets up the
callbacks for read/write readiness.

The read/write callbacks should be supplied by the caller, and will
call the `read` and `write` routines to write to buffers.

If `read` or `write` are NULL and exist in the event loop, the events
are removed.  This lets the callback move between event loops.

i.e. *all memory management is done by the caller*.

### debug

Takes a `fr_transport_t *` and ???, and prints debugging information about the method.

## Sample Configuration

````
listen NAME {
	transport = files # files, udp, tcp, tls, ...

	application = radius

	files {
		filename = detail

		load_factor = 10

		track = yes

	}

	udp {
		ipaddr = ...
		port = ...
		interface = ...
	}

	tcp {
		ipaddr = ...
		port = ...
		interface = ...

		tls = ${..tls}

		limit {
			...
		}
	}

	tls {
		

	}


	radius {
		clients = ...

		server = default

		type = Access-Request
		type = Status-Server

		Access-Request {
			... # any necessary config here
		}

	}
}

````


All application configuration is in the `application` reference.
There's probably not a better name.  Maybe `protocol`?

All of the application-specific configuration is in a section,
including clients, and which kind of packets it accepts.

## Startup

When the system starts up, it loads each listener and calls the parse
API.  It then does...

The transport bootstraps the IO layer, and calls any necessary
messaging API to insert the application into the worker threads.

The transport `read` function is called when the socket is ready.  The
`read` function in turn calls the underlying `io read` handler to read
from the socket.  When the transport function has determined it has a
packet, it performs any transport-specific processing.  e.g. RADIUS
de-dup, etc.

The transport `read` function then sends a message with both the
packet and the application layer to the worker thread, which processes
the request.
