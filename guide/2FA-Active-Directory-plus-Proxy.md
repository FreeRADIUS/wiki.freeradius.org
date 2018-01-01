# 2FA - AD password and External OTP via RADIUS proxy

### This guide is under construction...

This document describes how to set up FreeRADIUS to authenticate users in two steps. First the username/password is authenticated against Active Directory. If successful, an Access-Challenge message is returned to the client requesting it to send a second Access-Request with an OTP code. The second request is then proxied by FreeRADIUS to an external RADIUS OTP service for verification.

This guide was tested and verified using Gemalto Safenet Authentication Services (SAS) as the OTP service. However Gemalto SAS currently don't support pre-authenticating users AD-password before OTP, which is why I added a FreeRADIUS server in front of the SAS service to add AD-auth.

## Prerequisites

Install FreeRADIUS on your favourite Linux distribution. In this guide we have used CentOS 7, and FreeRADIUS v3.0.13 that is available in the CentOS repos:

<pre>
yum install -y freeradius freeradius-ldap freeradius-utils
</pre>

## FreeRADIUS Configuration

### LDAP Module

In this guide we'll use the LDAP module to perform AD authentication. To perform LDAP authentication against Active Directory, FreeRADIUS must know the users ClearText password, meaning the client must be configured to use PAP authentication. 

If you require supporting MS-CHAPv2 authentication, you should look into using Samba and winbind for authentication instead. Please see the following HOWTO:

[http://wiki.freeradius.org/guide/Active-Directory-direct-via-winbind](http://wiki.freeradius.org/guide/Active-Directory-direct-via-winbind)

Authenticating using LDAP can take a few different approaches:

1. Bind with an admin-user, perform a search for auth-user and return the Cleartext-Password. Verfify with PAP module.
2. Bind with an admin-user, perform a search for auth-user and then attempt to re-bind as authenticating user.
3. Attempt a direct bind as the authenticating user.

Method #1 doesn't work with Active Directory as the LDAP source as it doesn't allow you to poll user passwords, and #2 doesn't really gain us anything in this scenario, so in this guide we'll use method #3 which requires a minimal configuration and no admin/service-account is needed in the AD.

Edit `/etc/raddb/modules-available/ldap`:

<pre>
ldap {
        server = '<b>IP of Domain Controller</b>'
}
</pre>

### Proxy Realm

Edit `/etc/raddb/proxy.conf` and configure a <b>home_server</b>, <b>home_server_pool</b> and <b>realm</b>:

<pre>
home_server test {
        ipaddr          = 10.0.0.100
        port            = 1812
        secret          = testing123
        type            = auth
        src_ipaddr      = 10.0.0.10
}

home_server_pool test_pool {
        home_server     = test
}

realm proxy-test {
        auth_pool       = test_pool
}
</pre>

### Authentication / Authorization

Create a new site-file in `/etc/raddb/sites-available` or edit the `default` site:

<pre>
authorize {
        if (!State) {
                if (&User-Password) {
                        # If !State and User-Password (PAP), then force LDAP:
                        update control {
                                Ldap-UserDN := "%{User-Name}@my-domain.com"
                                Auth-Type := LDAP
                        }
                }
                else {
                        reject
                }
        }
        else {
                # If State, then proxy request:
                update control {
                        Proxy-To-Realm := "proxy-test"
                }
        }
}

authenticate {
        Auth-Type LDAP {
                # Attempt authentication with a direct LDAP bind:
                ldap
                if (ok) {
                        # Create a random <b>State</b> attribute:
                        update session-state {
                                State := "%{randstr:aaaaaaaaaaaaaaaa}"
                        }
                        update reply {
                                Reply-Message := "Please enter OTP"
                        }
                        # Return Access-Challenge:
                        challenge
                }
        }
}

pre-proxy {
        # Enable pre-proxy to filter State attribute from proxied requests:
        attr_filter.pre-proxy
}

</pre>

#### Configuration break-down

So, what is actually happening here? Lets break it down:

<pre>
        if (!State) {
</pre>

First we check if this is the first Access-Request packet we receive by checking for the existence of a <b>State</b> attribute. If it's absent that means this is the initial request.

<pre>
                if (&User-Password) {
</pre>

As we only support PAP authentication, if check for the existence of the <b>User-Password</b> attribute. If it's available we proceed:

<pre>
                                Ldap-UserDN := "%{User-Name}@my-domain.com"
</pre>

To force a direct LDAP bind using the authenticating user we explicitly set the <b>Ldap-UserDN</b> attribute. For Active Directory LDAP the syntax username@my-domain.com is usually working.

<pre>
                                Auth-Type := LDAP
</pre>

Force authentication to be done using LDAP Auth-Type.

<pre>
        else {
                # If State, then proxy request:
                update control {
                        Proxy-To-Realm := "proxy-test"
                }
</pre>

We did have a <b>State</b> attribute available meaning this is the second Access-Request we receive, so lets proxy it to our external OTP service provider by manually specifying the Realm we want to proxy it to.

<pre>
authenticate {
        Auth-Type LDAP {
                # Attempt authentication with a direct LDAP bind:
                ldap
                if (ok) {
                        # Create a random State attribute:
                        update session-state {
                                State := "%{randstr:aaaaaaaaaaaaaaaa}"
                        }
                        update reply {
                                Reply-Message := "Please enter OTP"
                        }
                        # Return Access-Challenge:
                        challenge
                }
        }
}
</pre>

In the <b>authenticate</b> section we send the request off to the <b>ldap</b> module for authentication by testing a direct bind using the credentials we received in the request. If it succeeds we create a new random <b>State</b> attribute and tell FreeRADIUS to return an <b>Access-Challenge</b> message to the client.

<pre>
pre-proxy {
        # Enable pre-proxy to filter State attribute from proxied requests:
        attr_filter.pre-proxy
}
</pre>

As our external OTP service provider only sees the second Access-Request message we need to filter out the <b>State</b> attribute from the proxied requests. We accomplish this by enabling `attr_filter.pre-proxy`.
