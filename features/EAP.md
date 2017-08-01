EAP Methods
===========

FreeRADIUS was the first Open Source RADIUS server to support EAP.
It has defined the standard for how RADIUS servers should manage
EAP sessions. As of Version 2.0, it supports more EAP methods than
**any other RADIUS server**, commercial or Open Source.

The following EAP methods are supported by FreeRADIUS 2.0 and later for
wired, or for [WiFi](WiFi) authentication.

Stable EAP Methods
------------------

The following EAP methods are considered "stable", and work with all
versions of FreeRADIUS.

EAP-GTC
EAP-MD5-Challenge
EAP-MSCHAPv2
EAP-PEAPv0
-   EAP-MSCHAPv2
-   EAP-GPSK
-   EAP-GTC
-   EAP-MD5-Challenge
-   EAP-TLS

EAP-SIM
EAP-TLS
-   *bare EAP-TLS*
-   EAP-TLS

EAP-TTLS
-   PAP
-   CHAP
-   MS-CHAP
-   MS-CHAPv2
-   EAP-MSCHAPv2
-   EAP-MD5
-   EAP-GTC
-   EAP-TLS

Old EAP Methods
---------------

The following EAP methods are distributed with the server, but should
not be used. They will likely be removed in a future version.

-   EAP-IKEv2

    *Not compatible with [RFC 5106](ietf.org/rfc/rfc5106.txt).*

-   Cisco LEAP

    *This method is insecure.*

-   EAP-TNC

    *Uses an old version of libtnc, and has not been tested in years.*

Version 2 EAP Methods
---------------------

The following EAP methods are considered "experimental" in Version 2.
They are implemented via the "eap2" module, by using the library
`libeap.so` from [hostapd](http://hostap.epitest.fi/hostapd/).

The "eap2" module was removed with the release of version 3.0.0. It was
experimental, and not well integrated with the rest of the server.

EAP-AKA
EAP-FAST
EAP-GPSK
EAP-PAX
EAP-PEAPv0
-   EAP-GPSK
-   EAP-PAX
-   EAP-PSK
-   EAP-SAKE

EAP-PEAPv1
-   EAP-GPSK
-   EAP-PAX
-   EAP-PSK
-   EAP-SAKE

EAP-PSK
EAP-SAKE
EAP-TTLS
-   EAP-MD5
-   EAP-GPSK
-   EAP-GTC
-   EAP-PAX
-   EAP-PSK
-   EAP-SAKE
