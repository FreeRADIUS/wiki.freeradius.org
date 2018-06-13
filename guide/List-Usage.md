# List usage

Originally, each list in the server is used for a specific purpose. As the server had grown in complexity, some lists have take on multiple roles.

This page documents the different roles, so that new modules can use lists consistently, and so users know where to find attributes.

The roles described below are valid for versions v4.0.x or greater, but were followed to a lesser extent in previous versions or the server.

## Attribute types
### Protocol attributes
Protocol attributes represent:
- Attribute Value Pairs (_AVPs_) from requests, such as ``&User-Name``.
- Attributes derived from fields in packets such as ``&DHCP-Gateway-Address`` (extracted from ``giaddr`` in DHCP packets).

These may be forwarded in proxied requests, or sent in response to requests (via the ``&reply:`` list).

### Internal attributes
These are created by the server's:
- Modules
- Core (state machine)
- Policies (unlang)

These will never be forwarded (except possibly in internal proxies), and never sent in responses.

## Lists and their uses
### ``&request:`` - Protocol Attributes
Holds the attributes created from an externally generated (by a client), or internally proxied request. 

### ``&reply:`` - Protocol Attributes
Holds the attributes to return in response to the requests. e.g. ``&reply:Session-Timeout``.

### ``&control:`` - Internal attributes (is never used for protocol attributes)
Holds attributes which alter the behaviour of modules, and 'check' attributes, which contain 
known good versions of things like User's passwords

### ``&session-state:`` - Internal attributes
The only list which persists between multiple requests.  It is linked by the ``&state`` attribute in the RADIUS protocol.

Attributes which need to be accessed between multiple requests, at an unknown or indeterminate time in an authentication session's life cycle should be inserted here.

A good example is ``TLS-Cert-*`` attributes, which are created during certificate validation, but used in multiple places during the lifetime of the request (mainly post-auth and the tls-verify virtual server).

### ``&reply:`` - Internal attributes
Attributes which control how the response will be sent e.g. ``&reply:FreeRADIUS-Response-Delay``

### ``&request:`` - Internal attributes
The result of deriving additional attributes from attributes in the request. e.g. When looking up a user's group membership, the resulting ``LDAP-Group`` attributes (if caching is enabled) are inserted into the request list.

The results of validation requests e.g. OCSP are also inserted into this list.

The results of operations (such as querying a HTTP server) are also inserted into this list.