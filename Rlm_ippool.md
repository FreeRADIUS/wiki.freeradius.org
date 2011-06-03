The '''rlm_ippool''' [[module]] helps manage [[IP Pool]]s.

{{Default in the server source|src/modules/rlm_ippool}}

## Default configuration

{{Default in the server source|raddb/modules/ippool}}

	#  Do server side ip pool management. Should be added in post-auth and
	#  accounting sections.
	#
	#  The module also requires the existance of the Pool-Name
	#  attribute. That way the administrator can add the Pool-Name
	#  attribute in the user profiles and use different pools
	#  for different users. The Pool-Name attribute is a *check* item not
	#  a reply item.
	#  The Pool-Name should be set to the ippool module instance name or to
	#  DEFAULT to match any module.
	#
	# Example:
	# radiusd.conf: ippool students { [...] }
	#		ippool teachers { [...] }
	# users file  : DEFAULT Group == students, Pool-Name := "students"
	#		DEFAULT Group == teachers, Pool-Name := "teachers"
	#		DEFAULT	Group == other, Pool-Name := "DEFAULT"
	#
	# ********* IF YOU CHANGE THE RANGE PARAMETERS YOU MUST *********
	# ********* THEN ERASE THE DB FILES                     *********
	#
	ippool main_pool {

		#  range-start,range-stop: The start and end ip
		#  addresses for the ip pool
		range-start = 192.168.1.1
		range-stop = 192.168.3.254

		#  netmask: The network mask used for the ip's
		netmask = 255.255.255.0

		#  cache-size: The gdbm cache size for the db
		#  files. Should be equal to the number of ip's
		#  available in the ip pool
		cache-size = 800

		# session-db: The main db file used to allocate ip's to clients
		session-db = ${raddbdir}/db.ippool

		# ip-index: Helper db index file used in multilink
		ip-index = ${raddbdir}/db.ipindex

		# override: Will this ippool override a Framed-IP-Address already set
		override = no

		# maximum-timeout: If not zero specifies the maximum time in seconds an
		# entry may be active. Default: 0
		maximum-timeout = 0

		# The key to use for the session database (which holds the allocated ip's)
		# normally it should just be the nas  ip/port (which is the default)
		#key = "%{NAS-IP-Address} %{NAS-Port}"
	}

## Custom configuration

        ippool main_pool {
                range-start = 192.168.10.1
                range-stop = 192.168.10.254
                netmask = 255.255.255.0
                cache-size = 254
                session-db = ${raddbdir}/db.ipmainpool
                ip-index = ${raddbdir}/db.ipmainindex
                override = no
                maximum-timeout = 0
        }
        ippool secondary_pool {
                range-start = 192.168.11.1
                range-stop = 192.168.11.254
                netmask = 255.255.255.0
                cache-size = 254
                session-db = ${raddbdir}/db.ipsecondarypool
                ip-index = ${raddbdir}/db.ipsecondaryindex
                override = no
                maximum-timeout = 0
        }

The two examples above simply show two pool entries. They differ slightly from the default config file.

You will notice that the two pool names are unique as are their corresponding db files. If the range of IP's change, the files must be deleted so they can be recreated on restart.

Also, note the cache size matches the number of IP's in your pool. More is OK but wasteful, less is very bad.

Override tells the server not to overwrite an existing entry - for instance if this pool is used in a group and the user has been previously assigned a static but will receive other attributes from a group configuration of which he might be a member.

At least one example of how to use the pools is located at [[ippool and radius clients]].

## See Also

* [[Ippool and radius clients]]
* [[rlm_ippool_tool]]
