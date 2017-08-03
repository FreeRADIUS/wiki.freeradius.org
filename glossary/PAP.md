**PAP** stands for Password Authentication Protocol.

It is the simplest authentication protocol used by FreeRADIUS, as
it requires only the Username and Password RADIUS
[attributes](attributes) (defined by the original RFCs) to
authenticate users.

By extension, it is also one of the least secure protocols,
because the password is protected **only** by the shared secret,
so anyone who is able to sniff RADIUS UDP datagrams between the
client and the server can pretty easily brute-force break that.

To use PAP with FreeRADIUS, you can match usernames with the
Cleartext-Password attribute in the [users](users) file, but also
using any other authentication back-end such as
[rlm\_ldap](rlm_ldap) or [rlm\_sql](rlm_sql).
