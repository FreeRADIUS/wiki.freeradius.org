The Microsoft Windows Operating System has a few special needs with respect to EAP methods that involve TLS, such as EAP-TLS and PEAP.  The most common indication that you have run into these issues is that the client sends a series of Access-Request messages, the server sends an series of Access-Challenge responses, and then... nothing happens. After a little wait, it all starts again.

#### Certificate Requirements

If you see this happening, then you have run into the Windows "special needs" problem.  The issue is that Windows looks for certain OIDs in the TLS certificate supplied by the RADIUS server.  If it does not see those OIDs, then it will simply drop the EAP conversation part way through the process.  There is very little you can do to solve the problem, other than meeting Windows "special needs".

See the following page on Microsoft's site for more information:

http://support.microsoft.com/kb/814394/en-us

If the clients are running Windows XP SP2, see also:

http://support.microsoft.com/kb/885453/en-us

You MUST follow the instructions on the first page, and install the hot fix from the second page for PEAP or EAP-TLS to work with a Windows machine.

In version 2.0 of the server, see also the "raddb/certs/" directory.  It includes scripts that will create certificates containing the correct information.  A normal installation of the server will create "test" certificates that also contain the correct information.

#### Root Certificates

Any "root" or CA certificate that is used to sign the server certificate must also be installed on the Windows machine.