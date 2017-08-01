Databases
=========

All common [databases](Databases) are
supported, as indicated below. For external (non-text) databases,
multiple databases can be used both for load-balancing and fail-over.
Databases can even be mixed across fail-over methods (e.g. if an LDAP
server is down, use an SQL server instead.)

Flat files
-   `users`
-   Realms
-   `/etc/passwd`
-   `/etc/group`
-   Any other `passwd` style file

LDAP
-   Active Directory (*for LDAP queries, or authentication via
    [samba](http://samba.org) and `ntlm_auth`*)
-   [OpenLDAP](http://openldap.org)
-   Novell eDirectory
-   Sun One Directory Server
-   [OpenDirectory](http://www.apple.com/server/macosx/features/directory.html)
-   Any LDAPv3 compliant directory

SQL
-   [Firebird](http://firebirdsql.org)[]()
-   [IODBC](http://www.iodbc.org)
-   MsSQL
-   [MySQL](http://mysql.org)
-   [Oracle](http://oracle.com)
-   [PostgreSQL](http://www.postgresql.org)
-   [UnixODBC](http://www.unixodbc.org)

[Policies](Policy) can be stored in any database, and policies can
selectively obtain information from any database.

Schemas and queries for both plain Internet access and VoIP are included
with the distribution. Schemas and queries for [IP pools](IPPool)
are also included.
