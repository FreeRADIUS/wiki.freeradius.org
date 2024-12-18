Authenticating against Active Directory using winbind
=====================================================

Many sites have Active Directory installed as their central user
directory. As such, wanting to authenticate against it from
FreeRADIUS is a common requirement. Unfortunately there are
several different ways to do this depending on the local
situation.

This cookbook recipe shows how to configure FreeRADIUS 3 to
authenticate MSCHAP against AD using winbind from the Samba
project. This will be of most use to those with wireless networks
that are using EAP methods such as PEAP/EAP-MSCHAPv2, which is
pretty much a given in an Active Directory environment for user
authentication (though this document does not go into the details
of configuring EAP).

_The method given in this document is new (March 2015) and may
require compiling code and may also contain bugs. However, in
testing, performance has been shown to be greater than other existing
methods such as ntlm_auth._

Other Active Directory authentication methods
---------------------------------------------

Most existing installations use **ntlm_auth and winbind**. See
[this
link](http://deployingradius.com/documents/configuration/active_directory.html),
where configuration examples are given for both PAP and MSCHAP
authentication.

This method is stable and is in production use many sites, but may
have performance issues once there are more than around 30
authentications per second.

It is also possible to use **LDAP** as an authentication backend
when using PAP, though this is not a recommended solution - LDAP
is a directory that can be used for authorization (such as group
lookup), and is not intended for authentication.


System Requirements
-------------------

The methods discussed in this document require at least the
following software versions:

* Samba 4.2.1 or greater
* FreeRADIUS 3.0.8 or greater


Compiling and installing Samba
==============================

If Samba 4.2.1 or greater has been released then either download
the official tarball, or install distribution packages.

Debian Jessie 8.4 now includes Samba 4.2.10. Instructions for
building Samba packages for Debian Jessie can be found on the
[[Debian package page|Debian#building-debian-samba-packages-with-threaded-libwbclient-support]]
if needed.

To compile Samba 4.2 and apply the correct patches use the steps
below. Alternatively, download the Samba source tarball and apply
[[the libwbclient-ctx.patch file|https://gist.github.com/mcnewton/71865fc18215c740b084]]. The patches
may apply to versions of Samba older than 4.2.0, but will not apply to Samba 3.

<pre>
git clone git://git.samba.org/samba.git
cd samba
git checkout samba-4.2.0
git checkout -b v4.2.0-wbclient
git cherry-pick 60c7571
git cherry-pick 83cfb84
git cherry-pick bc75e72
git cherry-pick 348f93f
git cherry-pick 063c56d
git cherry-pick 2664d90
git cherry-pick c6cb2d6
git cherry-pick 006da47
</pre>

If you choose to run with the latest Samba development codebase
then you can just checkout the master branch which already
includes the required patches. However, this is not recommended on
a production system.

Configure Samba with the following command. Note that the
configure script may stop and request that additional packages are
installed. You should be able to skip installing these packages by
passing the --without-* options presented to you.

For this example we will be installing into /opt/samba4.2. Using
-j on a multi-core machine will greatly speed up the compile time.

<pre>
./configure -C --prefix=/opt/samba4.2
make -j4
sudo make -j4 install
</pre>

Configuring Samba
-----------------

Samba now needs to be configured to join your domain. While this
is beyond the scope of this document, the following smb.conf may
help. If you installed as suggested above it will be in
`/opt/samba4.2/etc/smb.conf`:

<pre>
[global]
   netbios name = <b>THISSERVERNAME</b>
   workgroup = <b>WINDOWSDOMAIN</b>
   server string = RADIUS server
   security = ads
   invalid users = root
   socket options = TCP_NODELAY
   idmap uid = 16777216-33554431
   idmap gid = 16777216-33554431
   winbind use default domain = no
   winbind max domain connections = 5
   winbind max clients = 1000
   password server = *
   realm = <b>WINDOWSDOMAIN.EXAMPLE.COM</b>
</pre>

Update options in bold for your own site. Once smb.conf has been
created you should be able to join the server to the domain -
ensure that you use a domain account here that has permission to
create the machine account:

<pre>
sudo /opt/samba4.2/bin/net ads join -U Administrator
</pre>

The following should then return "Join is OK":

<pre>
sudo /opt/samba4.2/bin/net ads testjoin
</pre>

At this point, Samba has been configured and set up successfully.
You can test authentication with:

<pre>
/opt/samba4.2/bin/ntlm_auth --username=<b>user</b> --domain=<b>domain</b>
</pre>

Compiling and installing FreeRADIUS
===================================

Either download the FreeRADIUS 3.0.8 (or later) source, or
checkout from git:

<pre>
git clone git://github.com/FreeRADIUS/freeradius-server.git
cd freeradius-server
git checkout v3.0.x
</pre>

Configure the build process, making sure to refer to the
previously installed Samba location:

<pre>
./configure -C --disable-developer --prefix=/opt/fr3 --with-winbind-dir=/opt/samba4.2
</pre>

As part of the configure script you should see the following
lines:

<pre>
=== configuring in src/modules/rlm_mschap (.../freeradius-server/src/modules/rlm_mschap)
...
checking for wbclient.h in /opt/samba4.2/include/... yes
checking for wbcCtxAuthenticateUserEx in -lwbclient in /opt/samba4.2/lib... yes
</pre>

Which indicates that the Samba library has been found. Then build
and install:

<pre>
make -j4
sudo make install
</pre>

Configuring FreeRADIUS
----------------------

After install, the FreeRADIUS configuration should be in
`/opt/fr3/etc/raddb`. Uncomment the following options of the mschap
module configuration in `/opt/fr3/etc/raddb/mods-available/mschap`:

<pre>
winbind_username = "%{mschap:User-Name}"
winbind_domain = "<b>WINDOWSDOMAIN</b>"
</pre>

That's it - the rest of the default configuration should be fine.


Testing authentication
======================

First start up winbind. For testing it is recommended to run this
in the foreground in debug mode to see what is happening:

<pre>
sudo /opt/samba4.2/sbin/winbindd -SFd5
</pre>

In another terminal, start FreeRADIUS up:

<pre>
sudo /opt/fr3/sbin/radiusd -X
</pre>

And in a further terminal, test authentication using an active username
and password from your domain:

<pre>
/opt/fr3/bin/radtest -t mschap <b>username</b> <b>password</b> 127.0.0.1 0 testing123
</pre>

At which point you will hopefully see an Access-Accept. The
winbind and FreeRADIUS debug outputs should also confirm this.

Going into production
=====================

A note if running in production - once FreeRADIUS is configured
you are likely to want to run it as a non-privileged user. This
will mean that is is unable to access the winbind privileged
socket. If FreeRADIUS is running as user 'radiusd' which also has
primary group 'radiusd', then the following should fix the
directory permissions so that the socket can be accessed:

<pre>
sudo chgrp radiusd /opt/samba4.2/var/locks/winbindd_privileged
</pre>

There are only a few minor differences when running this code
rather than in the usual `ntlm_auth` configuration, which are how
authentications are logged in the debug output and the main
FreeRADIUS config file. Apart from this, authentications should be
faster and system load (especially fork rate) should decrease.
