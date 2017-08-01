VMPS
====

The VLAN Mangement Policy Server
([VMPS](http://en.wikipedia.org/wiki/VLAN_Management_Policy_Server))
system allows administrators to assign machines to a VLAN based on their
MAC address. The protocol itself was designed by Cisco, and is only
implemented on some Cisco switches.

FreeRADIUS added VMPS support in Release 2.0.0, and is currently the
only actively maintained Open Source VMPS server.

VMPS policies can be stored in flat-text files, and a sample policy is
shipped in the server as `raddb/sites-available/vmps`. The VLAN
assignments can also be stored in SQL, which simplifies the management
of thousands of devices.
