# Overview
This guide is intended to help any site wishing to join eduroam implement the IdP and SP eduroam components.  It contains sample configuration files that may be used in place of the normal v3.0.x configuration.

Logging, guest networks, and access point configuration are outside of the scope of this guide.

## Tooling
### eapol_test
Before you begin you should build a copy of ``eapol_test``.  ``eapol_test`` is an extremely useful tool produced by the hostapd project, which can simulate both a Wireless Access Point, and a Supplicant (client).  Unfortunately it's not usually packaged, and can be quite challenging to build.

As part of the FreeRADIUS CIT suite, we include a script to automatically build ``eapol_test``.  You should use this script.  Instructions are below.

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
### The outer virtual server ``sites-available/default``

**Replace the contents of this file.**

```text
operator_name = "<your-institutions-domain>"

server eduroam {
	listen {
		type = auth
		ipaddr = *
		port = 1812
	}

	authorize {
		# split_username_nai is a policy in the default distribution to 
		# split a username into username and domain.  We reject user-name 
		# strings without domains, as they're not routable.
		split_username_nai
		if (noop || !Stripped-User-Domain) {
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

		# If the EAP module returns 'ok' or updated, it means it has handled
		# the request and we don't need to call any other modules in this
		# section.
		eap {
			ok = return
			updated = return
		}
	}

	post-proxy {
		attr_filter
	}

	authenticate {
		eap
	}

	post-auth {
		linelog_success
		Poat-Auth-Type REJECT {
			linelog_failure
		}
	}
}
```