Freeradius as DHCP server for static ip allocation from MySQL, with
use of DHCP options.
Might work for provision of cable modems.
Example based on Debian.

One important thing! (specially Ubuntu users):  (*)
DHCP functionality will not work when installed from PPA, or at least
the package version 2.2.0 + dfsg-ppa10 not work.
Reason: I dont know. I installed and configured without errors, seems
to respond correctly OFFER and ACK but this packets never leaves the
network adapter.

==Before You Start==

This example assumes that:
        The network adapter where is connected freeradius have the following settings:

    auto eth1
    iface eth1 inet static
    address 192.168.10.1
    netmask 255.255.255.0
    network 192.168.10.0
    broadcast 192.168.10.255

    Client mac-address is 00:11:22:00:33:44


Let's start:

From sources:
- Download sources from Freeradius:
        wget ftp://ftp.freeradius.org/pub/freeradius/freeradius-server-2.2.0.tar.gz

- Unpack sources, then enter in new directory:
        tar -xvzf freeradius-server-2.2.0.tar.gz
        cd freeradius-server-2.2.0

- Configure it:
        ./configure --with-dhcp

- Add dictionary:           (**)
        edit share/dictionary (in sources files) and add a line containing
"$INCLUDE dictionary.dhcp" whithout quotes.

- If mysql is not installed, install it:
        apt-get install mysql-server

- Also, with mysql need some extra packages:    (***)
        apt-get install mysql-devel libmysqld-dev libmysqlclient-dev
libmysqld-dev libmysqld-pic

- Compile:
        make
        make install (do as root)

- Modify radiusd.conf (the configuration files is located in
/usr/local/etc/raddb)
        uncomment "$INCLUDE sql.conf"
        set "user = root" and "group = root"

- Modify sql.conf
        configure login/password for access to mysql database
        leave dialup.conf included

- Create database "radius"
        mysql -u user -p pass (login in mysql console)
                create database radius

- Load schema for mysql
        mysql -u user -p pass radius < schema.sql

- Add this in radius database:
        mysql -u user -p pass (login in mysql console)
                use radius;
                INSERT INTO `radcheck` (`username`, `attribute`, `op`, `value`)
VALUES ('00:11:22:00:33:44', 'Cleartext-Password', ':=', '');
                INSERT INTO `radreply` (`username`, `attribute`, `op`, `value`)
VALUES ('00:11:22:00:33:44', 'DHCP-Your-IP-Address', '=',
'192.168.10.10');

                        optionally included as example:

                INSERT INTO `radreply` (`username`, `attribute`, `op`, `value`)
VALUES ('00:11:22:00:33:44', 'DHCP-Subnet-Mask', '=',
'255.255.255.0');
                INSERT INTO `radreply` (`username`, `attribute`, `op`, `value`)
VALUES ('00:11:22:00:33:44', 'DHCP-Router-Address', '=',
'192.168.10.1');
                INSERT INTO `radreply` (`username`, `attribute`, `op`, `value`)
VALUES ('00:11:22:00:33:44', 'DHCP-Bootp-Extensions-Path', '=',
'modem.acf');
                INSERT INTO `radreply` (`username`, `attribute`, `op`, `value`)
VALUES ('00:11:22:00:33:44', 'DHCP-TFTP-Server-Name', '=',
'172.31.1.1');

- Modify /usr/local/etc/raddb/sql/dialup.conf, replace:
        sql_user_name = "%{User-Name}"
                for...
        sql_user_name = "%{DHCP-Client-Hardware-Address}"
                This use mac-address as username.

- create a /usr/local/etc/raddb/sites-enabled/dhcp_static and add this:
        (you can find the original example in
/usr/local/etc/raddb/sites-available/dhcp)

        server dhcp {
            listen {
                    type = dhcp
                    ipaddr = 255.255.255.255
                    port = 67
                    interface = eth1
                    broadcast = yes
            }

                dhcp DHCP-Discover {
                        update reply {
                               DHCP-Message-Type = DHCP-Offer
                        }

                        update reply {
                                DHCP-Domain-Name-Server = 0.0.0.0
                                DHCP-IP-Address-Lease-Time = 7200
                                DHCP-DHCP-Server-Identifier = 192.168.10.1
                        }

                        sql.authorize

                        ok
                }

                dhcp DHCP-Request {
                        update reply {
                               DHCP-Message-Type = DHCP-Ack
                        }

                        update reply {
                                DHCP-Domain-Name-Server = 0.0.0.0
                                DHCP-IP-Address-Lease-Time = 7200
                                DHCP-DHCP-Server-Identifier = 192.168.10.1
                        }

                        sql.authorize
                        sql.post-auth

                        ok
                }

                dhcp {
                        reject
                }

        }

- Start testing:
        /usr/local/sbin/radiusd -X


Troubleshooting:

- /usr/local/sbin/radiusd: error while loading shared libraries:
libfreeradius-radius-2.2.0.so: cannot open shared object file: No such
file or directory
        this solves the problem:
        /sbin/ldconfig -v

- Could not link driver rlm_sql_mysql: rlm_sql_mysql.so: cannot open
shared object file: No such file or directory
        Missing mysql devel extra packages. Install before compile!


clarifications:
        (*) - I installed PPA version in 3 different servers with no luck.
Same configuration, in sources version, works fine. Someone can
confirm this?
        (**) - In http://freeradius.org/features/dhcp.html says "un-comment",
but this line not exist.
        (***) -  installed (mysql-devel libmysqld-dev libmysqlclient-dev
libmysqld-dev libmysqld-pic) packages, surely one or more of them are
not necessary. If anyone knows which of them are the strictly
necessary, tell me, so I remove the rest.


TIP: You can use dhcpdump to see DHCP request and responses in your server.