# Cisco IOS and Radius
[[Cisco]] [[NAS]] equipment is quite popular, but being Cisco equipment running [[IOS]], the configuration can be a bit non-obvious to the unfamiliar. This document aims to describe the most common configuration options to make your Ciscos interoperate with [[RADIUS]] as you would expect a well-behaved [[NAS]] to do.

## Shell Access
To use [[RADIUS]] to authenticate your inbound shell (telnet & ssh) connections you need to create an entry in your users file similar to the following

    youruser   Cleartext-Password := "somepass"
               Service-Type = NAS-Prompt-User

This will let a user (called _youruser_) in for the first level of access to your Cisco. You will still need to **enable** to perform any configuration changes or anything requiring a higher level of access.

See [Configuring Basic AAA on an Access Server](http://www.cisco.com/en/US/tech/tk59/technologies_tech_note09186a0080093c81.shtml) for more details.

## Enable Mode

### Global Enable Password

When a shell user attempts to **enable** (or **enable 15**) on a cisco device, the [[cisco]] issues a [[RADIUS]] [[authentication]] request for the user **$enab15$**

If you type **enable 2**, it will send request for user '$enab2$', if you type **enable 3** it will send a request for '$enab3$' and so on.

These user(s) needs to to be configured on your [[RADIUS]] server with the password you wish to use to allow enable access.

    $enab15$   Cleartext-Password := "someadminpass"
               Service-Type = NAS-Prompt-User


### Per User Privilege Level

You can also send the privilege level (enable mode is level 15) for individual users as a reply item to automatically put them into that level with **cisco-avpair = "shell:priv-lvl=15"**

You can do this with an entry in your users file similar to the following

    youruser   Cleartext-Password := "somepass"
               Service-Type = NAS-Prompt-User,
               cisco-avpair = "shell:priv-lvl=15"

For more information, see Cisco page ["How to Assign Privilege Levels with TACACS+ and RADIUS"](http://www.cisco.com/en/US/tech/tk59/technologies_tech_note09186a008009465c.shtml).

### Command Authorization

Cisco claims that there is a complete mapping scheme to translate TACACS+ expressions into Cisco-AVPair Vendor-Specific. This works for example with the priv-lvl attribute:

               cisco-avpair = "shell:priv-lvl=15"

The two TACACS+ attributes "cmd" and "cmd-arg" would be needed for command authorization.There is a web page for [Cisco IOS](http://www.cisco.com/en/US/products/ps6350/products_configuration_guide_chapter09186a00804fe2d8.html) detailing which TACACS+ commands exist, and it suggests that

               cisco-avpair = "shell:cmd=show"

would do the trick to authorize the "show" command. EXCEPT that there is a tiny note for the commands "cmd" and "cmd-arg" saying that they cannot be used for encapsulation in the Vendor-Specific space.

These two are the ONLY ones. Since it's just about parsing the string content of cisco-avpair at the router side, there is absolutely no technical reason why these two wouldn't go through. The only explanation then is that this is a deliberate step by Cisco to make sure that TACACS+ is "superior" to RADIUS by arbitrarily cutting down functionality.

## IOS 12.x

For Cisco 12.x ( 12.0 and 12.1 ), the following AAA configuration directives are suggested:

    aaa new-model
    aaa authentication login default group radius local
    aaa authentication login localauth local
    aaa authentication ppp default if-needed group radius local
    aaa authorization exec default group radius local
    aaa authorization network default group radius local
    aaa accounting delay-start
    aaa accounting exec default start-stop group radius
    aaa accounting network default start-stop group radius
    aaa processes 6

this configuration works very well with most RADIUS servers.  One of the more important configurations is:

    aaa accounting delay-start

This directive will delay the sending of the Accounting Start packet until after an IP address has been assigned during the PPP negotiation process.  This will supersede the need to enable the sending of "Alive" packets as described below for IOS versions 11.x

## Common RADIUS Directives

IOS allows different ways of entering common config elements.  You can enter them as a single configuration stanza, or as separate items, as demonstrated by the two samples.

### Config Sample #1

The sample config below assumes two RADIUS servers with IP addresses 192.168.1.10 and 192.168.1.11. The sample specifies the RADIUS server and shared secret as a single config element, and it also sources all requests from interface Loopback0:
It also declares a group (named RadiusServers) and assign the two RADIUS servers to it. (Yes, you are essentially declaring the same server twice)

Note: Don't use the key of Cis$ko.  Make up your own.

    conf t
    aaa new-model
    radius-server host 192.168.1.10 auth-port 1812 acct-port 1813 key Cis$ko
    radius-server host 192.168.1.11 auth-port 1812 acct-port 1813 key Cis$ko

    ip radius source-interface Loopback0

    aaa group server radius RadiusServers
    server 192.168.1.10 auth-port 1812 acct-port 1813
    server 192.168.1.11 auth-port 1812 acct-port 1813
    exit

    aaa authentication login default group RadiusServers local
    exit

This sample assumes the password-encryption service is started on the device, so that the shared secrets will be encrypted after they’re entered. It is also highly recommended that a local login exist in case there is a failure to communicate with the RADIUS servers for any reason (the authentication order in the configlet specifies falling back to the local database after the RadiusServers group).

### Config Sample #2

The sample config below assumes two RADIUS servers with IP addresses 192.168.1.10 and 192.168.1.11. The sample specifies the RADIUS server and shared secret as a separate config elements.

Note: Don't use the key of Cis$ko.  Make up your own.

    conf t
    radius-server host 192.168.1.10
    radius-server key Cis$ko
    radius-server auth-port 1812
    radius-server host 192.168.1.11
    radius-server key Cis$ko
    radius-server auth-port 1812

### Shared Secret Encryption

IOS has a feature called the password-encryption service.

    service password-encryption
    no service password-encryption

From the ["Cisco Guide to Harden Cisco IOS Devices"](http://www.cisco.com/en/US/tech/tk648/tk361/technologies_tech_note09186a0080120f48.shtml#plane).

The actual encryption process occurs when the current configuration is written or when a password is configured. Password encryption is applied to all passwords, including username passwords, authentication key passwords, the privileged command password, console and virtual terminal line access passwords, and Border Gateway Protocol neighbor passwords. This command is primarily useful for keeping unauthorized individuals from viewing your password in your configuration file.

When password encryption is enabled, the encrypted form of the passwords is displayed when a more system:running-config command is entered.

**Caution This command does not provide a high level of network security. If you use this command, you should also take additional network security measures.**

Remember if your using password encryption, you **cannot** paste the encrypted password into the FreeRADIUS clients.conf file, It will not be the same shared secret.

## Nested Accounting

    aaa accounting nested

results in sending a second accounting start message, possibly causing problems with total usage counters.  Cisco NAS devices issue an Accounting Start packet when the user is authenticated, and again when a PPP session is initiated.  They send an Accounting Stop packet at the end of the PPP session, and a second at the conclusion of the call (usually nearly simultaneously).  Because of this, programs such as RadiusReport may see this as two connections, and would account for approximately twice the total time used.  Not using this nested command causes the NAS device to send an Accounting Stop packet followed almost immediately by an Accounting Start packet when a PPP connection is chosen, thereby eliminating the overlap.  This is particularly useful for those organizations interested in monitoring user usage accurately.
More information about this process can be seen [here](http://www.cisco.com/en/US/docs/ios/12_1/security/command/reference/srdacct.html#wp1022328).

## Unique Acct-Session-Id's

Minimum IOS: _12.2_
(Also available as a hidden command in _12.1(4.1)T_)

To enable RFC 2866 compliance (and stop duplicate _Acct-Session-Id_ values) on [[Cisco]] devices you need to issue the following command:

    radius-server unique-ident 1

Acct-Session-Id should now be unique for the next 256 reboots.

You must reboot after entering this command to take effect otherwise you will observe the following message after 10 minutes of entering this command:

    %RADIUS-3-IDENTFAIL: Save of unique accounting ident aborted.

## IOS 11.x

To get the For Cisco 11.1 to talk to a RADIUS server you normally use

    aaa new-model
    aaa authentication ppp radppp if-needed radius
    aaa authorization network radius none
    aaa accounting network wait-start radius


With IOS 11.3 if you want the IP address of the user to show up in the radutmp file (and thus, the output of [[radwho]]), you need to add

    aaa accounting update newinfo

Note: on newer ciscos you will need to use the command

    radius-server attribute 8 include-in-access-req

This is because with IOS 11.3, the Cisco first sends a "Start" accounting packet without the IP address included. By setting "update newinfo" it will send an account "Alive" packet which updates the information.

Also you might see a lot of "duplicates" in the logfile. That can be fixed by

    aaa accounting network wait radius
    radius-server timeout 3

## Ascend Style

To enable the Ascend style attributes (which we do **NOT** recommend!) add the **non-standard** keyword to your _radius-server_ line(s)

    radius-server host X.Y.Z.A auth-port 1812 acct-port 1813 non-standard

## Cisco VSAs

To see Cisco-AVPair attributes in the Cisco debugging log

    radius-server vsa accounting

## Static Loopback IP

The Cisco 36/26 by default selects (it seems at random) any IP address assigned to it (serial, ethernet etc.) as its RADIUS client source address, thus the access request may be dropped by the RADIUS server, because it can not verify the client. To make the cisco box always use one fixed address, add the following to your configuration:

    ip radius source-interface Loopback0

and configure the loopback interface on your router as follows:

    interface Loopback0
      ip address 192.168.0.250 255.255.255.255

Use a real world IP address and check the Cisco documentation for why it is a good idea to have working loopback interface configured on your router.

If you don't want to use the loopback interface of course you can set the source-interface to any interface on your Cisco box which has an IP address.

## Problems

According to [some reports](http://lists.freeradius.org/pipermail/freeradius-users/2006-November/014423.html), the Aironet 1200 series of access points works well, and fully supports RADIUS.

The Cisco WLC/WISM apparently use a single UDP socket for all RADIUS requests to a single server - auth and acct - and thus there's a 255-packet limit for in-progress requests. If the WLC reaches that limit, it just starts re-using IDs aggressively, instead of opening a socket, which is nice - if you're in the middle of processing a conflicted request, you still burn the work you're currently doing, and the result is never used.

This behavior causes issues during the traffic spikes. It is certainly a problem if you run an eduroam server, where proxied traffic can have very large RTTs. They're apparently going to "improve" this in 7.6 - there will be a separate UDP socket for auth/acct!

### Comments by the FreeRADIUS Team

The above behavior is *horrifically* bad.  Clients should open as many sockets as necessary to handle the load.  Using only one socket is a sign of laziness, and of bad design.  If the vendor doesn't fix basic RADIUS issues like this, we recommend people complain loudly, and then buy from another vendor.

## See Also

* [[HP]]
* [[Linksys]]
* [[VSA|Vendor-Specific Attributes]]
* [Cisco's Configuring AAA for Cisco Voice Gateways](http://www.cisco.com/en/US/docs/ios/voice/aaa/configuration/guide/va_aaa_overview_ps10591_TSD_Products_Configuration_Guide_Chapter.html)
* [Cisco's Configuring RADIUS with Livingston Server](http://www.cisco.com/en/US/tech/tk59/technologies_tech_note09186a008009467e.shtml)
* [Cisco Online Training Video](http://www.ciscotube.tk)
