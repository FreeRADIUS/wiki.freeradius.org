## Contents

 * [Plain Mac-Auth](#plain)
 * [Mac-Auth & 802.1x](#dot1x)

## Plain Mac-Auth {#plain}

This example assumes the server is only performing macauth. It checks MAC addresses against a [[users]] style file

#### raddb/policy.conf

NAS usually send the MAC address in the Calling-Station-ID attribute. There are several common formats:

 * 00:11:22:33:44:55
 * 00-11-22-33-44-55
 * upper-case hex
 * lower-case hex

It is sensible to re-format these into a single format at the server.

<pre>
#
# Rewrite called station id attribute into a standard format.
#
rewrite_calling_station_id {
        if (Calling-Station-Id =~ /([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})/i){
                update request {
                        Calling-Station-Id := "%{tolower:%{1}-%{2}-%{3}-%{4}-%{5}-%{6}}"
                }
        }
        else {
                noop
        }
}
</pre>

#### raddb/modules/file

<pre>
files authorized_macs {
        # The default key attribute to use for matches.  The content
        # of this attribute is used to match the "name" of the
        # entry.
        key = "%{Calling-Station-ID}"

        usersfile = ${confdir}/authorized_macs

        #  If you want to use the old Cistron 'users' file
        #  with FreeRADIUS, you should change the next line
        #  to 'compat = cistron'.  You can the copy your 'users'
        #  file from Cistron.
        compat = no
}
</pre>

#### raddb/authorized_macs

<pre>
00-11-22-33-44-55
  Reply-Message = "Device with MAC Address %{Calling-Station-Id} authorized for network access"
</pre>

#### raddb/sites-available/default

<pre>
authorize {
  preprocess

  # if cleaning up the Calling-Station-Id...
  rewrite_calling_station_id

  # now check against the authorized_macs file
  authorized_macs
  if (!ok) {
    reject
  }
  else {
    # accept
    update control {
      Auth-Type := Accept
    }
  }
}
</pre>

### Example 2 - macauth or 802.1x

This example shows how to mix 802.1x and macauth. The example does the following:

 1. If not using 802.1x, mac address must be known
 2. If using 802.1x, anyone with valid credentials can login (no mac address restrictions)

#### raddb/policy.conf

As per example 1

#### raddb/modules/file

As per example 1 

#### raddb/authorized_macs

As per example 1

#### raddb/sites-available/default

<pre>
authorize {
  preprocess

  # if cleaning up the Calling-Station-Id...
  rewrite_calling_station_id

  # If this is NOT 802.1x, assume mac-auth
  if (!EAP-Message) {
    # now check against the authorized_macs file
    authorized_macs
    if (!ok) {
      reject
    }
    else {
      # accept
      update control {
        Auth-Type := Accept
      }
    }
  }
  else {
    # normal FreeRadius virtual server config goes here e.g.
    eap
  }
}
</pre>

### Example 3 - macauth and 802.1x 

This example shows how to perform both 802.1x and macauth. The example does the following:

 1. If not using 802.1x, mac address must be known
 2. If using 802.1x, mac address must be known and valid credential given

#### raddb/policy.conf

As per example 1

#### raddb/modules/file

As per example 1 

#### raddb/authorized_macs

As per example 1

#### raddb/sites-available/default

<pre>
authorize {
  preprocess

  # if cleaning up the Calling-Station-Id...
  rewrite_calling_station_id

  # always check against the authorized_macs file first
  authorized_macs
  if (!ok) {
    reject
  }
  # If this is NOT 802.1x, mac-auth
  if (!EAP-Message) {
    # mac has already been checked, accept
    update control {
      Auth-Type := Accept
    }
  }
  else {
    # normal FreeRadius virtual server config goes here e.g.
    eap
  }
}
</pre>

### Example 4 - paranoid (checking the contents of the chap password attribute)

The examples below are a very rough example of how to perform MAC Based authentication with FreeRADIUS. You may need to add extra conditions to the authorize section, and perform additional canonicalization of the Calling-Station-ID and User-Name attributes, to make it work for your particular NAS.

#### raddb/policy.conf

As per example 1

#### raddb/modules/file

As per example 1 

#### raddb/authorized_macs

As per example 1

#### raddb/sites-available/default authorize{}

<pre>
#
# (Optional) May help if your NAS doesn't let you specify separators for the User-Name value
#

#rewrite_calling_station_id

#
# The EAP module should be listed before the Mac-Auth section if concurrent 802.1X/MAC authentication
# (Mac-Auth bypass etc...) is being used.
#
eap

#
# Machine (Calling-Station-ID based) authentication
#
# RFC 2865 says that a Service-Type value of Call Check is used
# to specify this kind of authentication (though were now dealing with ethernet ports instead of lines).
#
if((Service-Type == 'Call-Check') || (User-Name =~ /^%{Calling-Station-ID}$/i)){
        update control {
                Auth-Type = 'CSID'
        }
}

#
# If the CHAP module is called, it must be *after* the check for CSID authentication
#
# chap
</pre>

#### raddb/sites-available/default authenticate{}

<pre>
#
# Authentication based on Calling-Station-ID
#      
# Calling-Station-ID authentication is usually done by comparing normalised
# forms of the Calling-Station-ID and User-name fields.
#
Auth-Type CSID {
        #
        # Optionally a CHAP-Password attribute is included which is
        # md5(ChapID + Calling-Station-ID + Request Authenticator).
        #
        if(Chap-Password){
                update control {
                        Cleartext-Password := "%{User-Name}"
                }
                chap
        }
        else{
                ok  
        }  
}
</pre>

#### raddb/sites-available/default post-auth{}

<pre>
if(control:Auth-Type == 'CSID'){
        # Authorization happens here
        authorized_macs.authorize
        if(!ok){
                reject
        }
}
</pre>

## Additional modifications

### Mac-Auth authorisation by SSID

Follow any of the recipes above and then make the following modifications.

Note: The recipe below will work with any NAS that includes the SSID in the Called-Station-ID string with the format <BSSID>:<SSID> e.g.''"00-11-22-33-44-55:MY_SSID_1"''. There is no standard for this, and vendors may include the SSID in its own vendor specific attribute (VSA). If unsure, run the server in debug mode (-X) and check the contents of incoming requests.

If your vendor's NAS uses a VSA, omit the call to 'rewrite_called_station_id', do not add the additional attributes to the dictionary, and insert that VSA into the key attribute in place of '%{Called-Station-SSID}' e.g. ''key = "%{VENDOR_SSID_VSA}.%{Calling-Station-ID}"''.

#### raddb/dictionary

Use next free attribute number between 3000-4000 and insert the following definition.

<pre>
# The SSID the supplicant/user device connected to
ATTRIBUTE        Called-Station-SSID        3010                string
</pre>

#### raddb/policy.conf

Add the following policy stanza to policy.conf.

<pre>
#
# Rewrite called station id attribute into a standard format.
# If a 6th seperator is present, write the trailing chars into Called-Station-SSID
#
rewrite_called_station_id {
        if(Called-Station-Id =~ /^([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:.]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:.]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([-a-z0-9_.]*)?/i){
                update request {
                        Called-Station-Id := "%{1}%{2}%{3}%{4}%{5}%{6}"
                        Called-Station-SSID := "%{7}"
                }
        }
        else {
                noop
        }
}
</pre>

#### raddb/sites-available/default authorize{}

Add the a module call for 'rewrite_calling_station_id' to the authorize section directly above the call to 'rewrite_calling_station_id'.

<pre>
authorize {
  ...
  # break called station ID into the BSSID and SSID
  rewrite_called_station_id
  # if cleaning up the Calling-Station-Id...
  rewrite_calling_station_id
  ...
}
</pre>

#### raddb/modules/file

modify the key attribute of the authorized_macs files instance

<pre>
files authorized_macs {
        # The default key attribute to use for matches.  The content
        # of this attribute is used to match the "name" of the
        # entry.
        key = "%{Called-Station-SSID}.%{Calling-Station-ID}"

        usersfile = ${confdir}/authorized_macs

        #  If you want to use the old Cistron 'users' file
        #  with FreeRADIUS, you should change the next line
        #  to 'compat = cistron'.  You can the copy your 'users'
        #  file from Cistron.
        compat = no
}
</pre>

#### raddb/authorized_macs

Entries should now be in the following format.

<pre>
MY_SSID_1.00-11-22-33-44-55
    Reply-Message = "Device with MAC Address %{Calling-Station-Id} authorized for network access on SSID %{Called-Station-SSID}"
</pre>

### Mac-Auth authorisation by SSID SQL

#### raddb/dictionary

As above.
#### raddb/policy.conf

As above.
#### raddb/sites-available/default authorize{}

Add the a module call for 'rewrite_calling_station_id' to the authorize section directly above the call to 'rewrite_calling_station_id'.

<pre>

authorize {
    preprocess
    rewrite_calling_station_id
    rewrite_called_station_id
    if("%{sql:SELECT COUNT(*) FROM `SSIDMACAUTH` WHERE macaddress = '%{Calling-Station-ID}' AND SSID = '%{Called-Station-SSID}'}" >= 1){
          ok
        update control {
            Auth-Type := Accept
        }
    }
    else{
        reject
    }
}
</pre>
