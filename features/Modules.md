Modules
=======

The server is designed with a "plug-in"
[module](http://wiki.freeradius.org/Modules) system. Any feature that is
not needed can be removed from the configuration. Once it is removed
from the configuration, it does not affect server performance, memory
use, security, etc. This flexibility enables the server to run on
platforms ranging from embedded systems (e.g. LinkSys WRT54G) to
multi-core machines with gigabytes of RAM.

The module system permits simple integration of new features, as the
modules ave complete control over the processing of a packet. For
example, the [python](http://wiki.freeradius.org/Rlm_python) module adds
support for the Python language in less than 500 lines of C. The
[eap2](EAP) module adds support for more EAP types, in just over
500 lines of C.

On-line versions of the [man](http://freeradius.org/radiusd/man/) pages exist for most
modules.
