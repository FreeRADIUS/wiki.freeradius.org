# Lightweight Extensible Authentication Protocol

The _Lightweight Extensible Authentication Protocol_ (LEAP) is a proprietary wireless LAN authentication method developed by Cisco Systems.  Important features of LEAP are dynamic [[WEP]] keys and mutual [[authentication]] (between a wireless client and a [[RADIUS]] server). LEAP allows for clients to reauthenticate frequently; upon each successful authentication, the clients acquire a new WEP key (with the hope that the WEP keys don't live long enough to be cracked).

There is no native support for LEAP in any Windows operating system but is supported by third party [[supplicant]]s. The protocol has been known from the beginning to be vulnerable to dictionary attacks just like [[EAP-MD5]] but it wasn't until the release of [asleap](http://asleap.sourceforge.net) by Joshua Wright in 2003 that people began to argue that LEAP was a serious security liability. Cisco still maintains that LEAP can be secure if sufficiently complex passwords are used, but complex passwords are rarely used in the real world because of the difficulty they pose for average users.  Newer protocols like [[EAP-TTLS]] and [[PEAP]] do not have this problem because they create a secure [[TLS|Transport Layer Security]] tunnel for the [[MS-CHAPv2]] user authentication session and can operate on Cisco and non-Cisco Access Points.

Some 3rd party vendors also support LEAP through the [Cisco Compatible Extensions Program](http://www.cisco.com/web/partners/pr46/pr147/partners_pgm_concept_home.html).

## See Also
* [[EAP]]