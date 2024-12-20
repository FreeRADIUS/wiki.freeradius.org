Policies
========

Version 2.0 includes a powerful and advanced [policy
language](http://freeradius.org/radiusd/man/unlang.html). Policies
can be run at any stage during the handling of a request:

-   before authentication
-   during authentication
-   after authentication
-   before accounting
-   during accounting
-   before proxying
-   after proxying

Policies can be based on any attribute in a request or response, include
checking both the existence of (or lack of) an attribute, and the
contents of an attribute. They can filter out attributes, or re-write
the contents of attributes.

Attributes can be created, deleted, or edited in a policy. Policies can
leverage information in SQL, LDAP, flat-text files, or any other source
of data.

Policies can be based on identities (user, group, or role), location
(client IP, port, etc.), time (date, time of day), authentication method
(PAP, CHAP, MS-CHAP, EAP type, etc.), or any other piece of information
that is in a RADIUS packet or in a database. Policies can enforce VLAN
capabilities, filtering QoS, etc.
