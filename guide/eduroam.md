# Overview
This guide is intended to help any site wishing to join eduroam, implement the IdP and SP eduroam components.  It contains sample configuration files that may be used in place of the normal v3.0.x configuration.

Logging, guest networks, and access point configuration are outside of the scope of this guide.

## Tooling
### eapol_test
Before you begin you should build a copy of ``eapol_test`.  ``eapol_test`` is an extremely useful tool produced by the hostapd project, which can simulate both a Wireless Access Point, and a Supplicant (client).  Unfortunately it's not usually packaged, and can be quite challenging to build.

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


