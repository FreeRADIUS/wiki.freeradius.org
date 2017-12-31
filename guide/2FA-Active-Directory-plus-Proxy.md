# Introduction

This document describes how to set up FreeRADIUS server in order to facilitate 2FA where the initial request is authenticated against Active Directory and then proxied to an external RADIUS server for the second step.

# Authentication

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
