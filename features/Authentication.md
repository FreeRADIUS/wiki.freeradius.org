
Authentication
==============

The server can [authenticate](http://wiki.freeradius.org/Authentication)
users via PAP, CHAP,
[MS-CHAP](http://www.freeradius.org/rfc/rfc2548.txt),
[MS-CHAPv2](http://www.freeradius.org/rfc/rfc2548.txt), SIP Digest, and
all common [EAP](EAP) methods.

The ability to use a particular authentication protocol (PAP, CHAP,
types of [EAP](EAP)) is *completely* under the control of the
administrator. In most cases, the choice of authentication protocol is
under control of the user or NAS. However, the server does not have to
agree to accept that protocol. It can reject a request if it determines
that a protocol is insufficiently secure.

User passwords can be stored as clear-text, NT hashed (MD4), LM hashed,
MD5, salted-MD5, SHA1, salted-SHA1, or crypt (e.g. `/etc/passwd`). The
`pap` module understands passwords stored via the common LDAP method of
using a header, followed by password text. (e.g. `{header}text`). It
understands all commonly used header formats (`{clear}`, `{crypt}`,
etc.), and can parse the password text from clear-text, a hex string, or
a base-64 encoded string.

If a password is not available locally for some reason, the server can
pass the authentication to another system such as LDAP, PAM, Unix
(`/etc/passwd`), Kerberos, Active Directory, or RADIUS server via RADIUS
proxying. Local programs (e.g. CGI scripts) can also be used to
authenticate users via shell scripts or any other method.
[Perl](http://wiki.freeradius.org/Rlm_perl) or
[Python](http://wiki.freeradius.org/Rlm_python) scripts can be
pre-loaded into the server, which significantly lowers the cost of
running such programs.

The method used to obtain a "known good" password is completely
independent of the authentication protocol being used. A system can be
deployed using [EAP](EAP) for authentication, and can obtain
passwords from a flat-text file, LDAP, SQL, or even a Perl or Python
script.

All client operating systems are supported, including Windows XP (SP1
and SP2) and Vista, Linux, Mac OSX, \*BSD, and many others. Supported
device types include desktops, servers, embedded devices, portables
(phone, PDAs), etc.

With the wide deployment of the server, we can say with confidence
that *all available* devices and operating systems have been
tested.

