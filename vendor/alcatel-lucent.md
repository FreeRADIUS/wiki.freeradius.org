# Alcatel-Lucent and Radius

>
> This page is currently: "Work in Progress"
>

[Alcatel-Lucent Enterprise](http://enterprise.alcatel-lucent.com/) runs various product lines. In the beginning this page will focus on the details of OmniSwitch products.

## OmniSwitch
The Alcatel-Lucent OmniSwitch Vendor-Specific-Attributes (VSA) run under "Vendor ID" 800, hence you'll have to use the "XYLAN" dictionary.
The products run the "Alcatel-Lucent Operating System" (AOS) in two major release trees.

**AOS Release 6**
* AOS Release 6.4.x
 * OmniSwitch 6850E
 * OmniSwitch 6855
* AOS Release 6.6.x
 * OmniSwitch 6250
 * OmniSwitch 6450
 
**AOS Release 7**
* AOS Release 7.3.2.R01
 * OmniSwitch 6900
 * OmniSwitch 10K

### Authenticated Switch Access (ASA)

If you want to access the OmniSwitch via Telnet/SSH/WebView and authenticate the user via Radius, follow the steps below.

On the switch, configure the following ...
`-> aaa radius-server <radius_server_name> host <ipv4_address> key <secret>`
`-> aaa radius-server freeradius host 192.168.2.103 key verysecret`




***

Brainstorm:
Authorise the user access to the switch via RADIUS
Authorise MAC (non-supplicant)
Authorise 802.1x (supplicant)
Authorise Captive Portal
Bandwidth Management
service-type (call check, framed user)
radius test tool


> abc

`asdfasdf`

asdfasdfa

