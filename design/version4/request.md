# Things to track in the REQUEST

When the thread starts running a request, it makes a few checks to see
if the request *should* run.  i.e. is the exchange endpoint still
active?  If not, the reply can't go anywhere, and we should just stop
processing the packet.

For authentication packets, this is true.  For accounting packets, perhaps not.

* fr_time_t * for packet tracking, because the "original" packet can
  change underneath us.  We don't need a "context" pointer, or a function comparison pointer.

* `fr_exchange_t *` for the packet exchange, so that we can send replies up the same exchange point

* * maybe also a function pointer, so that the checks are protocol agnostic?

* * the packets come in with an an exchange endpoint, and we allocate the encapsulating structure ourselves.

* * the function pointer checks not only the exchange point, but also the underlying `fr_time_t *`

* thread-specific head / tail pointer for yielded sessions.

* next / prev pointers for yielded sessions

* `VALUE_PAIR* stats` for keeping statistics

