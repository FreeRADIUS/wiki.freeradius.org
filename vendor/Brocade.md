This article regards setting up freeradius auth on Brocade/Foundry hardware. Below example is on a MLX4 Netiron running OS Version 5.6.0T165. This procedure should be similar for most Brocade/Foundry close to 5.X version.

**Switch side config:**
```text
!set up local username/password for failover purposes
username <localusername> password <password>
! aaa-config
aaa authentication enable default radius local
aaa authentication login default radius local
aaa authentication login privilege-mode
radius-server host <ip> auth-port 1812 acct-port 1813 authentication-only key <KEY>
```

**User/group attribute configuration**
```text
! Set up the NAS as a normal NAS, and to give admin level privilege set below as reply attribute.
Foundry-Privilege-Level := 32768
```