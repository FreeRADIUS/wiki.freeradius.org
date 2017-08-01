DHCP
====

As of Version 2.2.0, the server has complete support for DHCP. It can
serve DHCP queries from an SQL databases, APIs, and various other data
stores. It can also act as a DHCP relay.

An example of a DHCP configuration is in `raddb/sites-available/dhcp`.
It should be edited for your local configuration.

Please Contribute
-----------------

Support for the DHCP protocol is faily stable. The examples work for
statically configured responses, and it is interoperable with all
clients that we have tested. At this time, it supports Option 82, but
not much else of the optional parts of the DHCP protocol. We are looking
for users/developers to assist with:

-   Additional testing.
-   Patches to add more functionality.
-   Modules to perform lease assignment using additional backends

Any assistance is useful. Please send email feedback to the
[freeradius-users](/list/users.html) list.
