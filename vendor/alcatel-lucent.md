# Alcatel-Lucent and Radius

>
> This page is currently: "Work in Progress"
>
> If you have questions or suggestions, you can reach me via Twitter: BennyE_HH
>

[Alcatel-Lucent Enterprise](http://enterprise.alcatel-lucent.com/) runs various product lines. In the beginning this page will focus on the configuration of/for OmniSwitch products.

## OmniSwitch
The Alcatel-Lucent OmniSwitch Vendor-Specific-Attributes (VSA) run as "Vendor ID" 800, hence you'll have to use the "XYLAN" dictionary.

The products run the "Alcatel-Lucent Operating System" (AOS) in two major release trees.

**Note:** Not all features are shared/available across the product lines, I'll do my best to pin-point what works in which release!

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

By default all access to the switch is limited to "console access" (User: admin, Password: switch)
Except for SSH if the switch obtained an IP address via DHCP, then SSH will be available for initial remote configuration (that is the case below).

    -> show aaa authentication 
    Service type = Default
      Authentication = denied
    Service type = Console 
      1st authentication server  = local
    Service type = Telnet
     Authentication = Use Default,
     Authentication = denied
    Service type = Ftp
     Authentication = Use Default,
     Authentication = denied
    Service type = Http
     Authentication = Use Default,
     Authentication = denied
    Service type = Snmp
     Authentication = Use Default,
     Authentication = denied
    Service type = Ssh
     1st authentication server  = local

**On the switch, configure the following ...**

    -> aaa radius-server freeradius host 192.168.2.103 key verysecret
    -> aaa authentication ssh freeradius local
    -> show aaa server freeradius
    Server name = freeradius
      Server type         = RADIUS,
      IP Address 1        = 192.168.2.103,
      Retry number        = 3,
      Time out (sec)      = 2,
      Authentication port = 1812,
      Accounting port     = 1813

Note: Please don't use "verysecret" as your key, it is just meant as a placeholder!

**On the Radius ...**

**/etc/freeradius/clients.conf**

    client 192.168.2.106/24 {
            secret          = verysecret
            shortname       = OmniSwitch
    }

**/etc/freeradius/users**

To create an admin account that has full read-write privileges is straight forward:

     admin           Cleartext-Password := "password"
                     Xylan-Asa-Access = "all",
                     Xylan-Acce-Priv-F-W1 = 0xFFFFFFFF,
                     Xylan-Acce-Priv-F-W2 = 0xFFFFFFFF

Now let us assume you want to create an "read-only" user that can see all aspects of the configuration.

(This is the **bad** example! Showcasing the most common mistake ...)

    readadmin       Cleartext-Password := "password"
                    Xylan-Asa-Access = "all",
                    Xylan-Acce-Priv-R1 = 0xFFFFFFFF,
                    Xylan-Acce-Priv-R2 = 0xFFFFFFFF

If you just use the configuration above, you'll see the following error message:

    Benny$ ssh readadmin@192.168.2.106
    readadmin's password for keyboard-interactive method: 
     
    PTY allocation request failed on channel 0

Instead the entry needs to look like this:

    readadmin       Cleartext-Password := "password"
                    Xylan-Asa-Access = "all",
                    Xylan-Acce-Priv-F-R1 = 0xFFFFFFFF,
                    Xylan-Acce-Priv-F-R2 = 0xFFFFFFFF,
                    Xylan-Acce-Priv-F-W1 = 0x00000002,
                    Xylan-Acce-Priv-F-W2 = 0x00000000

The result should look like this:

    Benny$ ssh readadmin@192.168.2.106
    readadmin's password for keyboard-interactive method: 
    
     
    Welcome to the Alcatel-Lucent OmniSwitch 6450
    Software Version 6.6.3.451.R01 Service Release, December 20, 2012. 
     
    Copyright(c), 1994-2012 Alcatel-Lucent. All Rights reserved.
    
    OmniSwitch(TM) is a trademark of Alcatel-Lucent registered
    in the United States Patent and Trademark Office.
  
    -> 
    -> vlan 20 enable 
    ERROR: Authorization failed. No functional privileges for this command
    -> show vlan 
                                  stree                 mble   src        
     vlan  type  admin   oper   1x1   flat   auth   ip   tag   lrn   name
    -----+-----+------+------+------+------+----+-----+-----+------+----------
       1    std   on     on     on    on     off    on   off     on   VLAN 1 
      10    std   on    off     on    on     off   off   off     on   VLAN 10
    
Although we just want to give "read-only" access to the OmniSwitch, we'll need to allow "read-write" for the access method, in this case SSH. The domains/command family values are simply added to the read-only or read-write entries.

You can get the values for the various commands/domains either via the CLI or by using the "Bitmap Calculator" that is accessible through the WebView of the AOS.

![webview.png](http://dokuwiki.alu4u.com/dokuwiki/lib/exe/fetch.php?t=1375813412&tok=845290&media=webview.png)
**TODO** ----- @freeRADIUS Administrator: Can we maybe host the pictures here? -----

     -> show aaa priv hexa ?
                     ^
                     WEBMGT VRRP VLAN UDLD TFTP-CLIENT TELNET SYSTEM STP SSH 
                     SNMP SESSION SCP-SFTP RMON RIP RDP QOS PORT-MAPPING 
                     POLICY PMM NTP NONE MODULE LOOPBACK-DETECTION LLDP 
                     LINKAGG IPV6 IPMS IPMR IP-ROUTING IP-HELPER IP INTERFACE 
                     HEALTH FILE DSHELL DOMAIN-SYSTEM DOMAIN-SERVICE 
                     DOMAIN-SECURITY DOMAIN-POLICY DOMAIN-PHYSICAL 
                     DOMAIN-NETWORK DOMAIN-LAYER2 DOMAIN-ADMIN DNS DHCP-SERVER 
                     DEBUG CONFIG CHASSIS BRIDGE AVLAN ALL AIP AAA 802.1Q <cr> 
     (AAA & Configuration Mgr Command Set)
    
    
    -> show aaa priv hexa ssh 
    0x00000002 0x00000000

    -> show aaa priv hexa telnet
    0x00000008 0x00000000

    -> show aaa priv hexa ssh telnet
    0x0000000a 0x00000000

The first value goes in R1/W1 and the second value in R2/W2, pretty easy (if you once figured it out).
The advantage is that at login the "allowed set of commands" is known by the switch without any further interaction with the Radius during the session.

As you can see above, you can do the calculation on the switch as well if you provide the commands/domains.

That said, a read-only user for all domains with SSH + Telnet access would look like this:

        readadmin       Cleartext-Password := "password"
                        Xylan-Asa-Access = "all",
                        Xylan-Acce-Priv-F-R1 = 0xFFFFFFFF,
                        Xylan-Acce-Priv-F-R2 = 0xFFFFFFFF,
                        Xylan-Acce-Priv-F-W1 = 0x0000000A,
                        Xylan-Acce-Priv-F-W2 = 0x00000000

Please don't forget that you'll have to add the following command on your OmniSwitch if you want Telnet to query the Radius.

    -> aaa authentication telnet freeradius local 

### Chapter 2
TODO

### Chapter 3
TODO

***

Brainstorm/TODO by Benny:
* **DONE:** Authorise the user access to the switch via RADIUS
* Authorise MAC (non-supplicant)
* Authorise 802.1x (supplicant)
* Authorise Captive Portal
* Bandwidth Management
* service-type (call check, framed user)
* radius test tool
* Remote-Configuration-Download
* Auth-Server-Down
* Unique Session ID for Accounting
* Accounting in general
* Filter-ID for UNPs
* MAC-ADDRESS format command switch for UPPER or lower case of MAc-ADddreSSes ;)
* User Community @ http://www.alcatelunleashed.com
* German DokuWiki @ http://dokuwiki.alu4u.com
