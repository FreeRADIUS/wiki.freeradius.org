The '''rlm_ldap''' FreeRADIUS module enables authentication via [[LDAP]].

To enable LDAP in your FreeRADIUS server, you can:

* instantiate an ldap module - which sets up the server name, the base DN, etc
* authenticate using an ldap module instance - which makes the FreeRADIUS server verify the user's identity in the LDAP directory, usually involving some form of checking the validity of the password
* authorize using an ldap module instance - which makes the FreeRADIUS server verify the user's level of authorization in the LDAP directory, usually involving verifying group membership or similar

# LDAP ATTRIBUTES

The mapping between RADIUS [[attributes]] and [[LDAP]] attributes is in raddb/ldap.attrmap. You can edit that file and add any new mapping that you may need. The LDAP-schema file is located in doc/RADIUS-LDAPv3.schema. Before adding any radius attributes the ldap server schema should be updated.

All ldap entries containing radius attributes should contain at least "objectclass: radiusprofile"

radiusCheckItem and radiusReplyItem are special. They allow the administrator to add any check or reply item respectively without adding it in the ldap schema. The format should be:
<pre>
 <ldap-attribute>: <radius-attribute> <operator> <value>
</pre>
For Example:
<pre>
 radiusReplyItem: Cisco-AVPair := "ip:addr-pool=dialin_pool"
</pre>

## CONFIGURATION

The following setup controls the rlm_ldap module.

<pre>
	# Lightweight Directory Access Protocol (LDAP)
	#  
	#  This module definition allows you to use LDAP for
	#  authorization and authentication.
	#
	#  See doc/rlm_ldap for description of configuration options 
	#  and sample authorize{} and authenticate{} blocks 
	#
	#  However, LDAP can be used for authentication ONLY when the
	#  Access-Request packet contains a clear-text User-Password
	#  attribute.  LDAP authentication will NOT work for any other
	#  authentication method.
	#
	#  This means that LDAP servers don't understand EAP.  If you
	#  force "Auth-Type = LDAP", and then send the server a
	#  request containing EAP authentication, then authentication
	#  WILL NOT WORK.
	#
	#  The solution is to use the default configuration, which does
	#  work.
	#
	#  Setting "Auth-Type = LDAP" is ALMOST ALWAYS WRONG.  We
	#  really can't emphasize this enough.
	#	
	ldap {
		server = "ldap.your.domain"
		#identity = "cn=admin,o=My Org,c=UA"
		#password = mypass
		basedn = "o=My Org,c=UA"
		filter = "(uid=%{Stripped-User-Name:-%{User-Name}})"
		#base_filter = "(objectclass=radiusprofile)"

		#  How many connections to keep open to the LDAP server.
		#  This saves time over opening a new LDAP socket for
		#  every authentication request.
		ldap_connections_number = 5

		timeout = 4
		timelimit = 3
		net_timeout = 1

		#
		#  This subsection configures the tls related items
		#  that control how FreeRADIUS connects to an LDAP
		#  server.  It contains all of the "tls_*" configuration
		#  entries used in older versions of FreeRADIUS.  Those
		#  configuration entries can still be used, but we recommend
		#  using these.
		#
		tls {
			# Set this to 'yes' to use TLS encrypted connections
			# to the LDAP database by using the StartTLS extended
			# operation.
			#			
			# The StartTLS operation is supposed to be
			# used with normal ldap connections instead of
			# using ldaps (port 689) connections
			start_tls = no

			# cacertfile	= /path/to/cacert.pem
			# cacertdir		= /path/to/ca/dir/
			# certfile		= /path/to/radius.crt
			# keyfile		= /path/to/radius.key
			# randfile		= /path/to/rnd
			# require_cert	= "demand"
		}

		# default_profile = "cn=radprofile,ou=dialup,o=My Org,c=UA"
		# profile_attribute = "radiusProfileDn"
		# access_attr = "dialupAccess"

		# Mapping of RADIUS dictionary attributes to LDAP
		# directory attributes.
		dictionary_mapping = ${raddbdir}/ldap.attrmap

		#  Set password_attribute = nspmPassword to get the
		#  user's password from a Novell eDirectory
		#  backend. This will work ONLY IF FreeRADIUS has been
		#  built with the --with-edir configure option.
		#
		# password_attribute = userPassword

		#  As of 1.1.0, the LDAP module will auto-discover
		#  the password headers (which are non-standard).
		#  It will use the following table to map passwords
		#  to RADIUS attributes.  The PAP module (see above)
		#  can then automatically determine the hashing
		#  method to use to authenticate the user.
		#
		#	Header		Attribute
		#	------		---------
		#	{clear}		User-Password
		#	{cleartext}	User-Password
		#	{md5}		MD5-Password
		#	{smd5}		SMD5-Password
		#	{crypt}		Crypt-Password
		#	{sha}		SHA-Password
		#	{ssha}		SSHA-Password
		#	{nt}		NT-Password
		#	{ns-mta-md5}	NS-MTA-MD5-Password
		#		
		#
		#  The headers are compared in a case-insensitive manner.
		#  The format of the password in LDAP (base 64-encoded, hex,
		#  clear-text, whatever) is not that important.  The PAP
		#  module will figure it out.
		#
		#  The default for "auto_header" is "no", to enable backwards
		#  compatibility with the "password_header" directive,
		#  which is now deprecated.  If this is set to "yes",
		#  then the above table will be used, and the
		#  "password_header" directive will be ignored.

		#auto_header = yes

		#  Un-comment the following to disable Novell
		#  eDirectory account policy check and intruder
		#  detection. This will work *only if* FreeRADIUS is
		#  configured to build with --with-edir option.
		#
		#edir_account_policy_check = no

		#
		#  Group membership checking.  Disabled by default.
		#
		# groupname_attribute = cn
		# groupmembership_filter = "(|(&(objectClass=GroupOfNames)(member=%{Ldap-UserDn}))(&(objectClass=GroupOfUniqueNames)(uniquemember=%{Ldap-UserDn})))"
		# groupmembership_attribute = radiusGroupName

		# compare_check_items = yes
		# do_xlat = yes
		# access_attr_used_for_allow = yes

		#
		#  By default, if the packet contains a User-Password,
		#  and no other module is configured to handle the
		#  authentication, the LDAP module sets itself to do
		#  LDAP bind for authentication.
		#
		#  You can disable this behavior by setting the following
		#  configuration entry to "no".
		#
		#  allowed values: {no, yes}
		# set_auth_type = yes
	}
</pre>

**NOTE**: As LDAP is case insensitive, you should probably also set "lower_user = yes" and "lower_time = before" in main section of radiusd.conf, to get limits on simultaneous logins working correctly. Otherwise, users will be able get large number of sessions, capitalizing parts of their login names.</p>

### LDAP Module Messages

On user rejection rlm_ldap will return the following module messages:
 "rlm_ldap: User not found" "rlm_ldap: Access Attribute denies access" 
 "rlm_ldap: Bind as user failed"

These messages will be visible in radius.log as additional information in "Login incorrect" and "Invalid user" log messages.

### LDAP xlat

The ldap module now supports LDAP URLs in xlat strings. That is you can now add LDAP URLs in the configuration options and hopefully shortly also in the users file. The strings will be of the following form:
<pre>
 %{ldap:ldap:///dc=company,dc=com?uid?sub?uid=%u}
</pre>
The requested attributes list MUST contain only ONE attribute. In case this attribute is multi valued which value is returned is considered UNDEFINED. Also, adding the host:port information SHOULD be avoided unless there are more than one ldap module instances in which case the host,port information can be used to distinguish which module will actually return the information (the xlat function will return NULL if the host,port information does not correspond to the configured attributes).  If there are more than one instances the module instance name can be used instead of the string 'ldap' before the ldap url to decide which instance will return the information. That is the xlat string will be of the form:
<pre>
 %{$instance_name:ldap:///dc=comapny,dc=com?uid?sub?uid=%u}
</pre>
For example:
<pre>
 %{ldap_company1:ldap:///dc=company1,dc=com?uid?sub?uid=%u}
</pre>

### User-Profile Attribute

The module can use the User-Profile attribute. If it is set, it will assume that it contains the DN of a profile entry containing radius attributes. This entry will _replace_ the default profile directive. That way we can use different profiles based on checks on the radius attributes contained in the Access-Request packets. For example (users file):
<pre>
DEFAULT Service-Type == Outbound-User, User-Profile := "uid=outbound-dialup,dc=company,dc=com"
</pre>

### Group Support

The module supports searching for ldap groups by use of the Ldap-Group attribute. As long as the module has been instanciated it can be used to do group membership checks through other modules. For example in the users file:

<pre>
DEFAULT Ldap-Group == "disabled", Auth-Type := Reject
  Reply-Message = "Sorry, you are not allowed to have dialup access"
</pre>

DNs are also accepted as Ldap-Group values, i.e.:

<pre>
DEFAULT Ldap-Group == "cn=disabled,dc=company,dc=com", Auth-Type := Reject
  Reply-Message = "Sorry, you are not allowed to have dialup access"
</pre>

Also if you are using multiple ldap module instances a per instance Ldap-Group attribute is registered and can be used. It is of the form &lt;instance_name&gt;-Ldap-Group. In other words if in radiusd.conf we configure an ldap module instance like:

<pre>
 ldap myname { [...] } 
</pre>

we can then use the myname-Ldap-Group attribute to match user groups. Make sure though that the ldap module is instantiated before the files module so that it will have time to register the corresponding attribute. One solution would be to add the ldap module in the instantiate{} block in radiusd.conf

### USERDN Attribute

When rlm_ldap has found the DN corresponding to the username provided in the access-request (all this happens in the authorize section) it will add an Ldap-UserDN attribute in the check items list containing that DN. The attribute will be searched for in the authenticate section and if present will be used for authentication (ldap bind with the user DN/password). Otherwise a search will be performed to find the user dn. If the administrator wishes to use rlm_ldap only for authentication or does not wish to populate the identity,password configuration attributes he can set this attribute by other means and avoid the ldap search completely. For instance it can be set through the users file in the authorize section:
<pre>
 DEFAULT Ldap-UserDN := `uid=%{User-Name},ou=people,dc=company,dc=com`
</pre>

### Directory Compatibility

If you use LDAP only for authorization and authentication (e.g. you can not afford schema extension), we suggest you set all necessary attributes in raddb/users file with following authorize section of radiusd.conf :
<pre>
 authorize {
  ldap {
    notfound = return
  }
  files
}

# See Also

* [[LDAP]]
