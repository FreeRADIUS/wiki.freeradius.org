# A very basic (but functional) Eduroam configuration
## Overview
This guide is intended to help any site wishing to join eduroam implement the IdP and SP eduroam components.  It contains sample configuration files that may be used in place of the normal v3.0.x configuration.

In addition to the configuration files here, you will need to configure a module to talk to your user store (LDAP, Novell, Active Directory, SQL).  See notes in the [inner-tunnel](#configuration_the-inner-virtual-server_sites-available-inner-tunnel) configuration.

## Tooling
### eapol_test
Before you begin you should build a copy of ``eapol_test``.  ``eapol_test`` is an extremely useful tool produced by the hostapd project which can simulate both a Wireless Access Point and a Supplicant (client).  Unfortunately it's not usually packaged and can be quite challenging to build manually.

As part of the FreeRADIUS CIT suite we include a script to automatically build ``eapol_test``.  You should use this script.

#### Dependencies
##### Centos/RHEL
```
sudo yum groupinstall "Development Tools"
sudo yum install git openssl-devel pkgconfig libnl3-devel
```
##### Ubuntu/Debian
```
sudo apt-get install git libssl-dev devscripts pkg-config libnl-3-dev libnl-genl-3-dev
```

#### Building
```
git clone --depth 1 --no-single-branch https://github.com/FreeRADIUS/freeradius-server.git

cd freeradius-server/scripts/travis/

./eapol_test-build.sh

cp ./eapol_test/eapol_test /usr/local/bin/
```

You should now have a functional ``eapol_test`` in your path.

## Configuration
You should replace existing configurations in their entirety with the configurations provided below.

### The outer virtual server

This is a bare bones configuration file for the outer virtual-server.  This virtual-server handles
the outer EAP conversation, i.e. data outside of the TLS tunnel.

It does the following:
- Checks the sanity of incoming ``User-Name`` strings.
- Routes requests that cannot be processed locally to the NRO (National Roaming Operator).
- Filters responses from remote IdPs.
- Calls the EAP module for authentication of local users.
- Assigns a VLAN based on whether the user is a local user, or was authenticated by a remote IdP.
- Logs authentication successes and failures.

***

#### ``sites-available/default``
```text
# The domain users will add to their username to have their credentials 
# routed to your institution.  You will also need to register this
# and your RADIUS server addresses with your NRO.
operator_name = "<your-institutions-domain>"

# The VLAN to assign eduroam visitors
eduroam_guest_vlan = "<guest-vid>"

# The VLAN to assign your students/staff
eduroam_local_vlan = "<local-vid>"

server eduroam {
	listen {
		type = auth
		ipaddr = *
		port = 1812
	}

	authorize {
		# Log requests before we change them
		linelog_recv_request

		# split_username_nai is a policy in the default distribution to 
		# split a username into username and domain.  We reject user-name 
		# strings without domains, as they're not routable.
		split_username_nai
		if (noop || !&Stripped-User-Domain) {
			reject
		}

		# Send the request to the NRO for your region
		# The details of the FLRs (Federation Level RADIUS servers).
		if (Stripped-User-Domain != ${operator_name}) {
			update {
				control:Proxy-To-Realm := 'eduroam_flr'
				
				# Operator name (RFC 5580) identifies the network the 
				# request originated from. It's not absolutely necessary
				# but it helps with debugging.
				request:Operator-Name := "1${operator_name}"
			}
			return
		}

		# If the EAP module returns 'ok' or 'updated', it means it has handled
		# the request and we don't need to call any other modules in this
		# section.
		eap {
			ok = return
			updated = return
		}
	}

	pre-proxy {
		attr_filter
		linelog_send_proxy_request
	}

	post-proxy {
		attr_filter
		linelog_recv_proxy_response
	}

	authenticate {
		eap
	}

	post-auth {
		# To implement eduroam you must:
		# - Use wireless access points or a controller which supports 
                #   dynamic VLAN assignments.
		# - Have that feature enabled.
		# - Have the guest_vlan/local_vlan available to the controller,
                #   or to all your access points.
		# Eduroam user traffic *MUST* be segregated, this is *NOT* optional.
		update reply {
			Tunnel-Type := VLAN
			Tunnel-Medium-Type := IEEE-802
		}
		if (&control:Proxy-To-Realm) {
			update reply {
				Tunnel-Private-Group-ID := ${eduroam_guest_vlan}
			}
		}
		else {
			update reply {
				Tunnel-Private-Group-ID := ${eduroam_local_vlan}
			}
		}

		# We're sending a response to one of OUR network devices for one of 
		# OUR users so provide it with the real user-identity.
		if (&session-state:Stripped-User-Name) {
			update reply {
				User-Name := "%{session-state:Stripped-User-Name}@%{Stripped-User-Domain}"
			}
		}

		linelog_send_accept

		Post-Auth-Type REJECT {
			attr_filter.access_reject
			linelog_send_reject
		}
	}
}
```
***

This is a stripped down EAP configuration that'll allow:
- EAP-TTLS-PAP
- EAP-TTLS-MSCHAPv2
- PEAPv0
- EAP-TLS

***
#### ``mods-available/eap``
```
eap {
	# The initial EAP type requested.  Change this to peap if you're
	# using peap, or tls if you're using EAP-TLS.
	default_eap_type = ttls

	# The maximum time an EAP-Session can continue for
	timer_expire = 60

	# The maximum number of ongoing EAP sessions
	max_sessions = ${max_requests}

	tls-config tls-common {
		# The public certificate that your server will present
		certificate_file = ${certdir}/server.pem

		# The private key for the public certificate
		private_key_file = ${certdir}/server.pem

		# The password to decrypt 'private_key_file'
		private_key_password = whatever

		# The certificate of the authority that issued 'certificate_file'
		ca_file = ${cadir}/ca.pem

		# If your AP drops packets towards the client, try reducing this.
		fragment_size = 1024

		# When issuing client certificates embed the OCSP URL in the 
		# certificate if you want to be able to revoke them later.
		ocsp {
			enable = yes
			override_cert_url = no
			use_nonce = yes
		}
	}

	tls {
		tls = tls-common
	}

	ttls {
		tls = tls-common
		default_eap_type = mschapv2
		virtual_server = "eduroam-inner"
	}

	peap {
		tls = tls-common
		default_eap_type = mschapv2
		virtual_server = "eduroam-inner"
	}
}
```

***

All Eduroam members should log requests to/from their servers for compliance purposes and because it makes debugging much easier.

These logging module instances and the virtual server configuration above will make your site fully compliant, so long as the syslog messages are retained for the requisite period.

We recommend ingesting the messages into logstash or Splunk to make debugging/helpdesk activities easier.

***

#### ``mods-available/linelog``
```
linelog linelog_recv_request {
	syslog_facility = local0
	syslog_severity = debug
	format = "action = Recv-Request, %{pairs:request:}"
}

linelog linelog_send_accept {
	syslog_facility = local0
	syslog_severity = debug
	format = "action = Send-Accept, %{pairs:request:}"
}

linelog linelog_send_reject {
	syslog_facility = local0
	syslog_severity = debug
	format = "action = Send-Reject, %{pairs:request:}"
}

linelog linelog_send_proxy_request {
	syslog_facility = local0
	syslog_severity = debug
	format = "action = Send-Proxy-Request, %{pairs:proxy-request:}"
}

linelog linelog_recv_proxy_response {
	syslog_facility = local0
	syslog_severity = debug
	format = "action = Recv-Proxy-Response, %{pairs:proxy-reply:}"
}
```

### The Inner virtual Server

This is a bare bones configuration file for the outer virtual-server.  This virtual-server handles
the inner EAP conversation, i.e. data inside of the TLS tunnel.

It does the following:
- Checks the sanity of the inner identity.
- Makes the inner identity available to the outer server.
- Retrieve's the users password/forwards credentials to ActiveDirectory (if used).
- Authenticates the user.

***

#### ``sites-available/inner-tunnel``
```
server eduroam-inner {
	listen {
		type = auth
		ipaddr = *
		port = 18120 # Used for testing only.  Requests proxied internally.
	}

	authorize {
		# The outer username is considered garabage for autz purposes, but 
		# the domain portion of the outer and inner identities must match.
		split_username_nai
		if (noop || !&Stripped-User-Domain || \
		    (&outer.Stripped-User-Domain != &Stripped-User-Domain)) {
			reject
		}

		# Make the user's real identity available to anything that needs
		# it in the outer server.
		update {
			&outer:session-state:Stripped-User-Name := &Stripped-User-Name
		}

		# EAP for PEAPv0 (EAP-MSCHAPv2)
		inner-eap {
			ok = return
			updated = return
		}

		# THIS IS SITE SPECIFIC
		#
		# The files module is *ONLY* used for testing.  It lets you define 
		# credentials in a flat file, IT WILL NOT SCALE.
		#
		# - If you use OpenLDAP with salted password hashes you should 
 		#   call the 'ldap' module here and use EAP-TTLS-PAP as your EAP method.
		# - If you use OpenLDAP with cleartext passwords you should 
		#   call the 'ldap' module here and use EAP-TTLS or PEAPv0.
		# - If you use an SQL DB with salted password hashes you should call 
		#   the 'sql' module here and use EAP-TTLS-PAP as your EAP method.
		# - If you use an SQL DB with cleartext passwords you should call 
		#   the 'sql' module here and use EAP-TTLS or PEAPv0.
		# - If you use Novell you should call the 'ldap' module here and 
		#   set ``edir = yes`` in ``mods-available/ldap`` and use EAP-TTLS or
		#   PEAPv0.
		# - If you use Active Directory, you don't need anything here (remove 
		#   the call to files) but you'll need to follow this 
		#   [guide](freeradius-active-directory-integration-howto) and use 
		#   EAP-TTLS-PAP or PEAPv0.
		# - If you're using EAP-TLS (i'm impressed!) remove the call to files.
		#
		# EAP-TTLS-PAP and PEAPv0 are equally secure/insecure depending on how the 
		# supplicant is configured. PEAPv0 has a slight edge in that you need to 
		# crack MSCHAPv2 to get the user's password (but this is not hard).
		files

		pap
		mschap
	}

	authenticate {
		inner-eap
		mschap
		pap

		# Comment pap, and uncomment the stanza below if you're using 
		# Active Directory this will allow it to work with EAP-TTLS-PAP.
		#pap {
		#	ntlm_auth
		#}
	}
}
```

***

This is a stripped inner-eap configuration that'll allow
- EAP-MSCHAPv2 (inner of PEAPv0)

***

#### ``mods-available/inner-eap``

```
eap inner-eap {
	default_eap_type = mschapv2
	timer_expire = ${modules.eap.timer_expire}
	max_sessions = ${modules.eap.max_sessions}

	mschapv2 {
		send_error = yes
	}
}
```