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
