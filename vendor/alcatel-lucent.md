# Alcatel-Lucent and Radius

>
> This page is currently: "Work in Progress"
>

[Alcatel-Lucent Enterprise](http://enterprise.alcatel-lucent.com/) runs various product lines. In the beginning this page will focus on the configuration for OmniSwitch products.

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

On the switch, configure the following ...

    -> aaa radius-server freeradius host 192.168.2.103 key verysecret
    -> aaa authentication ssh freeradius local
    xyu

On the Radius ...

clients.conf

client 192.168.2.106/24 {
        secret          = verysecret
        shortname       = OmniSwitch
}

users

Let us assume you want to create a "read-only" user that can see all aspects of the configuration.

(This is the the bad example!)

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
    



***

Brainstorm:
Authorise the user access to the switch via RADIUS
Authorise MAC (non-supplicant)
Authorise 802.1x (supplicant)
Authorise Captive Portal
Bandwidth Management
service-type (call check, framed user)
radius test tool
Remote-Configuration-Download
Auth-Server-Down


> abc

`asdfasdf`

asdfasdfa

