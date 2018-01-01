# 2FA - AD password and External OTP via RADIUS proxy

This document describes how to set up FreeRADIUS to authenticate users in two steps. First the username and password is authenticated against their password in Active Directory. If successful, an Access-Challenge message is returned to the client requesting it to provide an OTP code. The second request from the client containing the OTP is then proxied by FreeRADIUS to an external RADIUS OTP service for verification.

This guide was tested and verified using Gemalto Safenet Authentication Services (SAS) as the OTP service. However Gemalto SAS currently don't support pre-authenticating users AD-password before OTP, which is why I added FreeRADIUS server in front of the SAS service to add AD-auth.


## Authentication

```text 
authorize {
        if (!State) {
                update control {
                        Ldap-UserDN := "%{User-Name}@mydomain.com"
                        Auth-Type := LDAP
                }
        }
        else {
                update control {
                        Proxy-To-Realm := "test"
                }
        }
}

authenticate {
        Auth-Type LDAP {
                ldap-test
                if (ok) {
                        update session-state {
                                State := "%{randstr:aaaaaaaaaaaaaaaa}"
                        }
                        update reply {
                                Reply-Message := "Please enter OTP"
                        }
                        challenge
                }
        }
}

```