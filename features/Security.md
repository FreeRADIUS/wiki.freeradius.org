Security
========

A RADIUS server is a critical part of a network security system. Any
successful attack on a RADIUS server means that anyone can be
authenticated, with potentially catastrophic consequences. Two
requirements are necessary for any secure server; secure source code,
and secure configuration.

The Source Code is Secure
-------------------------

The [security](http://freeradius.org/security/) page contains the
public record of security issues in FreeRADIUS. (Try asking a
commercial vendor for *that* information!) The source code is
freely available, including the complete history of all changes.
Our policy is to be transparent and open about all security issues
with the server.

While this policy may seem to open the server to attacks from
people looking at the source code, the reality is different. That
openness has permitted us to be part of the [Coverity
scan](http://scan.coverity.com/) project for many years. *All* of
the issues found by [Coverity](http://coverity.com/) have been
fixed. That is, the code has been scanned for a large class of
potential issues, and has been proven to not be vulnerable to
those issues.

"Fail-Safe" Configuration
-------------------------

The server configuration is designed to be "fail-safe". The default
configuration requires administrator edits before any user can be
authenticated. The default configuration and internal policy is to
reject all requests that have not successfully been authenticated.
