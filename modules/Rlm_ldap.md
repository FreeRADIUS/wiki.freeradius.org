The *rlm_ldap* FreeRADIUS module enables authentication via [[LDAP]].

To enable LDAP in your FreeRADIUS server, you can:

* instantiate an ldap module - which sets up the server name, the base DN, etc
* authenticate using an ldap module instance - which makes the FreeRADIUS server verify the user's identity in the LDAP directory, usually involving some form of checking the validity of the password
* authorize using an ldap module instance - which makes the FreeRADIUS server verify the user's level of authorization in the LDAP directory, usually involving verifying group membership or similar

# LDAP ATTRIBUTES

In version 2, the mapping between RADIUS [[attributes]] and [[LDAP]] attributes is in raddb/ldap.attrmap. You can edit that file and add any new mapping that you may need. The LDAP-schema file is located in doc/schemas/ldap/openldap/freeradius.schema. Before adding any radius attributes the ldap server schema should be updated.

All ldap entries containing radius attributes should contain at least "objectclass: radiusprofile"

radiusControlAttribute and radiusReplyAttribute are special. They allow the administrator to add any control or reply item respectively without adding it in the ldap schema. The format should be:
<pre>
 ldap-attribute: radius-attribute operator value
</pre>

The version 3 attribute mapping is in the module configuration file `raddb/mods-available/ldap`

For Example:
<pre>
 radiusReplyAttribute: Cisco-AVPair := "ip:addr-pool=dialin_pool"
</pre>

### LDAP Module Messages

On user rejection rlm_ldap will return the following module messages:

* rlm_ldap: User not found
* rlm_ldap: Access Attribute denies access
* rlm_ldap: Bind as user failed

These messages will be visible in radius.log as additional information in "Login incorrect" and "Invalid user" log messages.

### LDAP xlat

The ldap module now supports LDAP URLs in [[xlat strings|Rlm_expr]]. That is you can now add LDAP URLs in the configuration options and hopefully shortly also in the users file. The strings will be of the following form:
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

The module supports searching for ldap groups by use of the Ldap-Group attribute. As long as the module has been instantiated it can be used to do group membership checks through other modules. For example in the users file:

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

You can also define the logic around LDAP groups inside the "post-auth" section in the sites-available/default file. The following example rejects the auth attempt if the user does not belong to either "LDAP Group One" or "MyGroupTwo".

<pre>
post-auth {
        if (LDAP-Group == "LDAP Group One") {
                noop
        }
        elsif (LDAP-Group == "MyGroupTwo") {
                noop
        }
        else {
                reject
        }
}
</pre>

By default, the LDAP group search will be done using the group's Common Name, and by appending the **groupmembership_filter** value as a search filter.  

Modify the **groupmembership_filter** according to what will return a unique group membership from your specific LDAP server.  For example, a groupmembership_filter for a Windows 2008 R2 Active Directory server could be: 
<pre>groupmembership_filter = "(|(&(objectClass=group)(member=%{control:Ldap-UserDn})))"</pre>

Using the filter above, a group search for user "CN=test,CN=Users,DC=domain,dc=com" (which was populated from %{control:Ldap-UserDn}) and group "MyGroup" (taken from the LDAP-Group attribute) would yield a final search filter of:
<pre>"(&(cn=MyGroup)(|(&(objectClass=group)(member=CN\3dtest\2cCN\3dUsers\2cDC\3ddomain\2cDC\3dcom))))"</pre>  

If the ldap module returns <pre>rlm_ldap::ldap_groupcmp: Group MyGroup not found or user is not a member error</pre> then use ldapsearch with the final filter to troubleshoot why the group search returned no result (could a bad filter, no actual membership, searching for the wrong LDAP attribute, etc).  

Example ldapsearch command to verify group attributes (Windows 2008 R2 Active Directory target):   
<pre>ldapsearch -x -b "cn=Users,dc=domain,dc=com" -D "cn=test,cn=Users,dc=domain,dc=com" -h domain.com -w "password" "(&(cn=MyGroup))"</pre>

Note: In 2.x.x the module LDAP-Group was associated with, was largely random and dependent on module instantiation order. In 3.x.x LDAP-Group will always refer to the ``ldap {}`` instance.

### USERDN Attribute

When rlm_ldap has found the DN corresponding to the username provided in the access-request (all this happens in the authorize section) it will add an Ldap-UserDN attribute in the check items list containing that DN. The attribute will be searched for in the authenticate section and if present will be used for authentication (ldap bind with the user DN/password). Otherwise a search will be performed to find the user dn. If the administrator wishes to use rlm_ldap only for authentication or does not wish to populate the identity,password configuration attributes he can set this attribute by other means and avoid the ldap search completely. For instance it can be set through the users file in the authorize section:
<pre>
 DEFAULT Ldap-UserDN := "uid=%{User-Name},ou=people,dc=company,dc=com"
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
</pre>

### Errors with LDAP over TLS connections

If your secure LDAP connection inexplicably fails with one or more of the below messages in the debug log, you are probably using a version of libldap that has been built with NSS:
<pre>TLS: could not shutdown NSS - error -8053:NSS could not shutdown. Objects are still in use..
rlm_ldap (ldap): Opening additional connection (7), 1 of 32 pending slots used
rlm_ldap (ldap): Connecting to ldap://ldap.example.com:636
TLS: could not find the slot for the certificate '/etc/raddb/certs/ldap-ca.pem' - error 
-8127:The security card or token does not exist, needs to be initialized, or has been removed..
TLS: /etc/raddb/certs/ldap-ca.pem is not a valid CA certificate file - error -8127:The 
security card or token does not exist, needs to be initialized, or has been removed..
TLS: could not perform TLS system initialization.
TLS: error: could not initialize moznss security context - error -8127:The security card or 
token does not exist, needs to be initialized, or has been removed.
TLS: can't create ssl handle.
rlm_ldap (ldap): Bind with cn=Radius,o=Example,c=XX to ldap://ldap.example.com:636 failed:
Can't contact LDAP server
TLS: could not shutdown NSS - error -8053:NSS could not shutdown. Objects are still in use..
rlm_ldap (ldap): Opening connection failed (7)
(28)     [ldap] = fail</pre>

Consider switching to a version of libldap that uses OpenSSL. FreeRADIUS uses OpenSSL everywhere, and OpenSSL and NSS don't play well together. You may wish to encourage your distribution's vendor to switch their libldap implementation (or make an OpenSSL-built version of libldap available) by raising a bug report with them.

In FreeRADIUS 3.x, you can set the **uses**, **lifetime** and **idle_timeout** settings in the **pool** section of the LDAP module to **zero** to keep the LDAPS connections open permanently to avoid this issue. 

This problem was visible mainly with official RedHat packages, because they switched to NSS. Fortunately, this problem has been fixed in RHSA-2017:1852 / https://access.redhat.com/errata/RHSA-2017:1852

**Updating package to openldap-2.4.44-5.el7.x86_64 solve this problem.**

# See Also

* [[LDAP]]