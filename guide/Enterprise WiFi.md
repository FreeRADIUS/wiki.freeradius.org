Enterprise Wi-Fi / IEEE 802.1X
==============================

Deficiencies of PSK networks
----------------------------

Common home-use Wi-Fi networks do not need a RADIUS server because they
"secure" the network with **one single network key**, the "WPA/WPA2
Pre-Shared Key" (PSK). That key is the same for every user, is often
guessable, and **can't be revoked for one user** (if one user should be
denied access, the key needs to be changed for the entire network and
every user needs to be given the new key). When a network is sniffed, an
attacker can perform **offline attacks** to guess the key.

**All this makes PSK networks unfit for enterprise use.**

IEEE 802.1X and RADIUS Authentication
-------------------------------------

The IEEE standards for Wi-Fi (IEEE 802.11) foresee an "Enterprise" mode
which is fundamentally different from PSK networks because the Wi-Fi
encryption keys are provisioned **per user** and **per session**. Every
user needs to authenticate with their **personal credentials**; at that
moment a key is generated and is communicated to the user’s device and
the NAS they connect to.

Before users send their authentication credentials, the **the user must
authenticate the network**, proving that it is indeed genuine; only then
is the client’s credential released. The IEEE standard IEEE 802.1X
(using RADIUS and the Extensible Authentication Protocol, EAP) is used
for authentication and key management.

Enterprise Wi-Fi authentication also enables advanced features such as
putting users dynamically into a specific VLAN (e.g. separate guest and
staff logins into different IP networks even though being on the same
SSID), and dynamic ACLs

Enterprise Wi-Fi requires:

1.  A RADIUS server which can do EAP authentication.
2.  Wi-Fi equipment which is correctly configured to use
    RADIUS authentication.
3.  user devices configured to do Enterprise Wi-Fi correctly.

RADIUS Server
-------------

The central component in an IEEE 802.1X / Enterprise Wi-Fi environment
is the RADIUS server: it receives RADIUS packets from the Wi-Fi Access
Point / Controller (see below), processes those by either proxying it to
another server (in a roaming environment) or by processing the packet
and authenticating the user itself.

To authenticate the user, the RADIUS server extracts the EAP
authentication data from the EAP-Message attribute of the RADIUS packet
and acts on the contents - It takes the role of an EAP server.

EAP authentication typically involves establishing a TLS tunnel with a
server certificate (i.e. the user authenticating the server) and then
exchanging user credentials (i.e. the user authenticating to the
server). After the authentication with EAP, the RADIUS server adds VLAN
assignment attributes and other authorisation data, and sends those in
the final RADIUS packet.

FreeRADIUS excels not only at RADIUS, but also in its role as EAP
server. It supports a wide variety of EAP authentication methods and
allows sending additional authorisation data. It supports RADIUS
proxying of EAP authentication traffic and thus enables roaming; and it
supports altering authorisation parameters after the initial
authentication has taken place (CoA).

Wi-Fi
-----

The access point / Wi-Fi controller needs to be capable of the WPA2
Enterprise operation mode; configuration options in the device are often
called IEEE 802.1X Authentication, RADIUS or WPA Enterprise. In this
operation mode, the device becomes a NAS (i.e. RADIUS client), so needs
to be configured with the IP address of the RADIUS server to connect to
and the corresponding shared secret for the RADIUS communication as
configured on the RADIUS server.

The aforementioned per-user VLAN assignment is not part of IEEE 802.1X;
it is an optional feature described by [RFC
3580](https://tools.ietf.org/html/rfc3580), and may not be available on
your particular controller. If you are provisioning devices and find
this feature valuable, you should look for features called "Dynamic VLAN
Assignment" "VLAN Override" or similar. To make use of the feature, the
last RADIUS packet sent from the RADIUS Server to the Access Point, the
Access-Accept packet, needs to include VLAN assignment attributes.

User Device Configuration
-------------------------

After having completed the RADIUS server and Wi-Fi equipment setup, it
is *possible* for users to authenticate securely to the network and to
verify that they are not connected to an attacker network ("evil twin").

For this possibility to actually happen, the network operator needs to
communicate the pertinent RADIUS server settings to all end users, and
they need to ensure that the end users transform these server-side
settings into a proper client-side configuration.

This typically involves:

-   Import or mark as trusted the CA certificate to validate the RADIUS
    server's certificate.
-   Configure the expected server name of the server certificate.
-   Select an EAP type on the client device which matches one of the
    configured EAP types on the server.
-   Optionally enable extra features such as enhanced privacy protection
    (use of anonymous outer identities) etc.

Network administrators should be aware that neglecting to secure the
client-side setup puts the security of the enterprise network at risk -
badly configured user devices can be tricked to connect to a rogue AP /
evil twin network! Such an attacher network can learn their credentials
as they authenticate - and the attacker can from then on log into the
genuine network with those same credentials. The attacker can also
inspect their payload - with the false feeling of being connected to the
own enterprise network, some users or their applications may use
unencrypted communication where they should not.

Manual setup instructions in e.g. PDF "click here and there"
instructions have proven to be error-prone, time-consuming and typically
trigger significant helpdesk activity.

There are automatic deployment tools for various platforms on the
market; e.g the "Apple Configurator" application can produce
configuration files for iOS devices and Mac OS computers.

There are also web services which cater for a variety of platforms at
the same time. Many of those are commercial, but some provide their
services for free or freemium. One such example is the [Enterprise
Network Configuration Assistant Tool (CAT)](https://802.1x-config.org)
at <https://802.1x-config.org> - all basic end-user device provisioning
functionality on all supported platforms (Windows, Mac OS, iOS, Linux)
is free; extra features such as custom branding or digitally signed
installation programs are a paid-for extra.

If your interest in reading this page is because you are part of the
"eduroam" roaming consortium: you get a full-service client-side
installer generator (and more!) at [eduroam
CAT](https://cat.eduroam.org).
