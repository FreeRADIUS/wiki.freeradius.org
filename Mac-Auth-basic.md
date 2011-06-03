#### raddb/policy.conf

You may (optionally) wish to rewrite the Calling-Station-ID attribute, to remove, or add, a standard set of separators between the octets of the Mac-Address.

<pre>
#
# Rewrite called station id attribute into a standard format.
#
rewrite_calling_station_id {
        if(request:Calling-Station-Id =~ /([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})[-:]?([0-9a-f]{2})/i){
                update request {
                        Calling-Station-Id := "%{1}-%{2}-%{3}-%{4}-%{5}-%{6}"
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
