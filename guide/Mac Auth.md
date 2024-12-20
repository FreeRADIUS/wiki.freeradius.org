## Plain Mac-Auth

This first example assumes the server is only performing mac-auth.
It checks MAC addresses against a [[users]] style file.

### raddb/policy.conf

Most NASes usually send the MAC address in the Calling-Station-ID attribute.
There are several common formats:

 * 00:11:22:33:44:55
 * 00-11-22-33-44-55
 * 0011.2233.4455

Again, depending on the NAS, these can be either upper-case or lower-case hex.

It is sensible to re-format these into a single format at the server. The
following policy is available in FreeRADIUS version 3 onwards, in
`[[raddb/policy.d/canonicalization|https://github.com/FreeRADIUS/freeradius-server/blob/v3.0.x/raddb/policy.d/canonicalization`.

<pre>
#
# Rewrite called station id attribute into a standard format.
#
rewrite_calling_station_id {
        if (Calling-Station-Id =~ /([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:.]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:.]?([0-9a-f]{2})[-:]?([0-9a-f]{2})/i) {
                update request {
                        Calling-Station-Id := "%{tolower:%{1}-%{2}-%{3}-%{4}-%{5}-%{6}}"
                }
        }
        else {
                noop
        }
}
</pre>

### raddb/mods-available/files

Create a new instance of the files module to read a new file of permitted MAC addresses.

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

### raddb/authorized_macs

This is the list of permitted MAC addresses, as read by the new files configuration above.

<pre>
00-11-22-33-44-55
        Reply-Message = "Device with MAC Address %{Calling-Station-Id} authorized for network access"
</pre>

### raddb/sites-available/default

Finally, call the canonicalisation policy and new files module from the authorize section:

<pre>
authorize {
        preprocess

        # If cleaning up the Calling-Station-Id...
        rewrite_calling_station_id

        # Now check against the authorized_macs file
        authorized_macs

        if (!ok) {
                # No match was found, so reject
                reject
        }
        else {
                # The MAC address was found, so update Auth-Type
                # to accept this auth.
                update control {
                        Auth-Type := Accept
                }
        }
}
</pre>

## Mac-Auth or 802.1x

This example shows how to mix 802.1x and mac-auth. The example does the following:

 1. If not using 802.1x, mac address must be known;
 2. If using 802.1x, anyone with valid credentials can login (no mac address restrictions).

The files `raddb/policy.conf`, `raddb/mods-available/files` and `raddb/authorized_macs`
are the same as the plain mac-auth examples above.

### raddb/sites-available/default

<pre>
authorize {
        preprocess

        # If cleaning up the Calling-Station-Id...
        rewrite_calling_station_id

        # If this is NOT 802.1x, assume mac-auth. We check this by testing
        # for the presence of the EAP-Message attribute in the request.
        if (!EAP-Message) {
                # Now check against the authorized_macs file
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
                # Normal FreeRADIUS virtual server config goes here e.g.
                eap
        }
}
</pre>

## Mac-Auth and 802.1x 

This example shows how to perform both 802.1x and mac-auth. The example does the following:

 1. If not using 802.1x, mac address must be known
 2. If using 802.1x, mac address must be known and valid credential given

The files `raddb/policy.conf`, `raddb/mods-available/files` and `raddb/authorized_macs`
are the same as the plain mac-auth examples above.

### raddb/sites-available/default

<pre>
authorize {
        preprocess

        # If cleaning up the Calling-Station-Id...
        rewrite_calling_station_id

        # always check against the authorized_macs file first
        authorized_macs

        if (!ok) {
		# Reject if the MAC address was not permitted.
                reject
        }

        # If this is NOT 802.1x, mac-auth
        if (!EAP-Message) {
                # MAC address has already been checked, so accept
                update control {
                        Auth-Type := Accept
                }
        }
        else {
                # Normal FreeRADIUS virtual server config goes here e.g.
                eap
        }
}
</pre>

## Web-Auth safe Mac-Auth

Although this configuration is more complex, you should probably use it if the server is going to process both web-auth and mac-auth requests, here is the rationale:

* Some NAS vendors allow both Web-Auth and Mac-Auth to occur on the same NAS on the same port, and do not provide attributes to distinguish between the two.
* This allows users to enter a username and password in the format of a Mac-Address and the RADIUS server would assume the NAS was requesting Mac-Auth.
* This makes Mac-Spoofing even more trivial as the Mac-Address of the NIC doesn't need to be overridden (not every OS/NIC supports this).
* Where a site implements Web-Auth for guest wireless connections, and Mac-Auth for wired connections, it allows malicious users to get wireless access by using Mac formatted credentials (If the policy does not check NAS-Port-Type).

This configuration attempts to prevent this kind of spoofing:

* Checks for the presence of a ``Service-Type == 'Call-Check'`` AVP as an explicit indication that the NAS wants to do Mac-Auth. If your NAS sends this in Access-Request packets, you should remove the ``User-Name =~ /^%{Calling-Station-ID}$/i`` sub-condition from the authorize section.
* Verifies that the CHAP-Password attribute matches the Calling-Station-ID of the station - this prevents users from spoofing macs via the web form. 

#### Note
For this configuration to work, you must configure the password format for Mac-Auth to use the same octet separator as the Calling-Station-ID attribute.

### raddb/policy.conf

As per example 1

### raddb/modules/file

As per example 1 

### raddb/authorized_macs

As per example 1

### raddb/sites-available/default

<pre>
authorize {
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
}

authenticate {
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
                        Cleartext-Password := "%{Calling-Station-ID}"
                    }
                    chap
            }
            else {
                ok  
            }  
    }
}

post-auth {
    if(control:Auth-Type == 'CSID'){
        # Authorization happens here
        authorized_macs.authorize
        if(!ok){
                reject
        }
    }
}
</pre>

## Additional modifications

### Mac-Auth authorisation by SSID

Follow any of the recipes above and then make the following modifications.

Note: The recipe below will work with any NAS that includes the SSID in the Called-Station-ID string with the format **<BSSID>:<SSID>** e.g. ``00-11-22-33-44-55:MY_SSID_1``. There is no standard for this, and vendors may include the SSID in its own vendor specific attribute (VSA). If unsure, run the server in debug mode (-X) and check the contents of incoming requests.

If your vendor's NAS uses a VSA, omit the call to 'rewrite_called_station_id', do not add the additional attributes to the dictionary, and insert that VSA into the key attribute in place of '%{Called-Station-SSID}' e.g. ``key = "%{VENDOR_SSID_VSA}.%{Calling-Station-ID}"``.

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
        if(Called-Station-Id =~ /^([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:.]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:.]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([-a-z0-9_. ]*)?/i){
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

#### raddb/sites-available/default authorize

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
#### raddb/sites-available/default authorize

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
