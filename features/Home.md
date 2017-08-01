Features
========

The server has all of the features that are found in a normal RADIUS
server implementation. It is unique among Open Source RADIUS servers in
it's support for [EAP](EAP). In addition, it has many capabilities
not found in any other RADIUS products, even commercial servers from
large vendors. In spite of that complexity, it is [simple](Simple)
to install and configure.

In short, if you need a certain feature, it is likely that FreeRADIUS
supports it. Please see the text below for information about specific
features.

Performance and Scalability
---------------------------

The server is one of the [fastest](Fast) and
[scalable](Scalability) products we know of. The unique
[modular](Modules) design enables it to be stripped down for
embedded systems, or to expose all of the available features where
required.

In addition to being simple, FreeRADIUS is designed to be
[secure](Security). An AAA server is a critical part of a network,
and a primary target for attackers. Keeping the server safe and secure
is a high priority for us, and for any administrator using the product.

Operating Systems
-----------------

The server runs on a wide variety of [Operating Systems](OS). Every
effort is make to ensure it is 64-bit "clean", though occasional issues
are discovered during development.

A large number of CPU and OS architectures are have been verified
to work, and are supported via the [users
list](http://freeradius.org/list/users.html). We suggest that the
server be installed via a [pre-built package](OS) if one is
available. If a package does not exist for your system, it can be
built from [source](http://freeradius.org/download.html).

AAA Functionality
-----------------

### Authentication

The server can [authenticate](Authentication) users via simple
methods (PAP, CHAP,
[MS-CHAP](http://www.freeradius.org/rfc/rfc2548.txt),
[MS-CHAPv2](http://www.freeradius.org/rfc/rfc2548.txt), SIP Digest) and
all common [EAP](EAP) types. All client operating systems are
supported, including Windows XP (SP1 and SP2) and Vista, Linux, Mac OSX,
\*BSD, and many others.

### Authorization

Both [pre-authentication](Pre-Auth) and
[post-authentication](Post-Auth) [policies](Policy) are
supported. Polices may be stored in [databases](Databases),
flat-text files, or in
[Perl](http://wiki.freeradius.org/modules/Rlm_perl) or
[Python](http://wiki.freeradius.org/modules/Rlm_python) scripts.

IP addresses can be allocated through [IP Pools](IPPool).

### Accounting

All common [accounting](Accounting) methods are supported.
Accounting data can be logged to flat-text files (`detail`), or to most
[databases](Databases). Schemas and queries for both plain Internet
access and VoIP are included.

### Proxying

Any RADIUS request can be [proxied](Proxy). Standard RADIUS realms
are supported via simple configurations. More complex policies can use
any method necessary to cause a request to be proxied.

Vendor Dictionaries
-------------------

Over 100 [vendor dictionaries](Vendors) are supported. All common
vendor equipment is supported, including all common attributes used by
each vendor. Vendor updates can be mailed to <dictionary@freeradius.org>

Databases
---------

All commonly used [databases](Databases) are supported for
authorization, authentication, and accounting. (LDAP, SQL, text files,
etc.). Fail-over and load-balancing across multiple servers is also
supported.

Virtual Servers
---------------

FreeRADIUS is the only RADIUS server (commercial or Open Source) that
supports [virtual servers](Virtual Servers). This feature is
similar to the virtual servers used in well-known web servers such as
Apache. This feature can simplify complex implementations, and can
reduce ongoing support and maintenance costs.

Other functionality
-------------------

VLAN assignment may be done via the [VMPS](VMPS) protocol. IP
address assignment can be done via the experimental [DHCP](DHCP)
implementation.

Specifications
--------------

All RADIUS [RFC's](http://freeradius.org/rfc/) are supported. The server is compliant with
the following specifications:

[rfc1157](http://ietf.org/rfc/rfc1157.txt)
[rfc1227](http://ietf.org/rfc/rfc1227.txt)
[rfc1448](http://ietf.org/rfc/rfc1448.txt)
[rfc1901](http://ietf.org/rfc/rfc1901.txt)
[rfc1905](http://ietf.org/rfc/rfc1905.txt)
[rfc2243](http://ietf.org/rfc/rfc2243.txt)
[rfc2289](http://ietf.org/rfc/rfc2289.txt)
[rfc2433](http://ietf.org/rfc/rfc2433.txt)
[rfc2548](http://ietf.org/rfc/rfc2548.txt)
[rfc2607](http://ietf.org/rfc/rfc2607.txt)
[rfc2618](http://ietf.org/rfc/rfc2618.txt)
[rfc2619](http://ietf.org/rfc/rfc2619.txt)
[rfc2620](http://ietf.org/rfc/rfc2620.txt)
[rfc2621](http://ietf.org/rfc/rfc2621.txt)
[rfc2716](http://ietf.org/rfc/rfc2716.txt)
[rfc2759](http://ietf.org/rfc/rfc2759.txt)
[rfc2809](http://ietf.org/rfc/rfc2809.txt)
[rfc2865](http://ietf.org/rfc/rfc2865.txt)
[rfc2866](http://ietf.org/rfc/rfc2866.txt)
[rfc2867](http://ietf.org/rfc/rfc2867.txt)
[rfc2868](http://ietf.org/rfc/rfc2868.txt)
[rfc2869](http://ietf.org/rfc/rfc2869.txt)
[rfc2882](http://ietf.org/rfc/rfc2882.txt)
[rfc2924](http://ietf.org/rfc/rfc2924.txt)
[rfc3162](http://ietf.org/rfc/rfc3162.txt)
[rfc3575](http://ietf.org/rfc/rfc3575.txt)
[rfc3576](http://ietf.org/rfc/rfc3576.txt)
[rfc3579](http://ietf.org/rfc/rfc3579.txt)
[rfc3580](http://ietf.org/rfc/rfc3580.txt)
[rfc3748](http://ietf.org/rfc/rfc3748.txt)
[rfc4372](http://ietf.org/rfc/rfc4372.txt)
[rfc4675](http://ietf.org/rfc/rfc4675.txt)
[rfc4679](http://ietf.org/rfc/rfc4679.txt)
[rfc4818](http://ietf.org/rfc/rfc4818.txt)
[rfc4849](http://ietf.org/rfc/rfc4849.txt)
[rfc5080](http://ietf.org/rfc/rfc5080.txt).

In addition to implementing standards, FreeRADIUS is
defining new industry standards for RADIUS.

The server has been tested to be [interoperable](Interoperability)
with a wide range of clients, servers, and 802.1X supplicants.
