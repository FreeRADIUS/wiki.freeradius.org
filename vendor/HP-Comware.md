
### HP A5500 - Comware 5.20 and 7.

**Switch side config:**
```text
system-view
! Configure the RADIUS Scheme!
radius scheme freeradius-scheme
 server-type extended
 primary authentication <primary_radius_server_ip>
 primary accounting <primary_radius_server_ip>
_! Optional: If Secondary RADIUS Server is planned and operational_
_ !secondary authentication <secondary_radius_server_ip>
 !secondary accounting <secondary_radius_server_ip>_
 key authentication <radius_key>
 key accounting <radius_key>
 user-name-format without-domain
! Note: This is the source IP connecting to the RADIUS Server(s)
 nas-ip <network_device_management_ip like a loopback>
!
! Configure the Domain!
domain freeradius-domain
 authentication login radius-scheme freeradius-scheme
 authorization login radius-scheme  freeradius-scheme
 accounting login radius-scheme radius-scheme
 access-limit disable
 state active
 idle-cut disable
 self-service-url disable
!
! Apply scheme to the remote access terminals

user-interface vty 0 15
 undo user privilege level 
 authentication-mode scheme
!
! WARNING: Ensure RADIUS server is working properly prior activating this!
domain default enable radius-domain
```

**Server-side setup:**
```text
client <network_device_management_ip> {
secret = <radius_key>
nastype = cisco
shortname = <network_device_name>
}
```

```text
**User/group attribute configuration**
netadmin Cleartext-Password := "netadmin"
Service-Type = NAS-Prompt-User,
!NOTE: RADIUS Attribute between 0-3, 0 visitor, 1 monitor, 2 system, 3 manager.
Huawei-Exec-Privilege = "3",
!Login-Service 50 is for SSH, Login-Service 0 for Telnet, but works without defining this
!Login-Service = 50
```

```text
**If running Comware 7**
Still on user/group attribute settings
netadmin Cleartext-Password := "netadmin"
    Service-Type = Administrative-User,
    !RADIUS Attribute for original H3C/Comware
    Huawei-Exec-Privilege = "3",
    Login-Service = 50,
    Cisco-AVPair = "shell:roles=\"network-admin\""
```