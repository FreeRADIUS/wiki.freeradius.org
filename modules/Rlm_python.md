## Python module for freeradius

### Purpose
To allow module writers to write modules in a high-level language, for implementation or for prototyping.

### Gotchas
The module rlm_python isn't built by default in FreeRADIUS. To include it you should use the following command line:

 ``./configure --with-experimental-modules``
