Hello all,

I am having a problem concerning Mikrotik RouterOS. I have to authenticate my clients over a FreeRadius server. This server has Simultaneous-Use set to 1.

The problem is that when my mikrotik pppoe nas reboots (like energy drop or something like that) this do not send ANY information about AcctStop to FreeRadius so, as you can see, the clients keep connected to FreeRadius.

What can I do to solve this problem? I tryed checkrad but it will not work becouse Mikrotik é "other" to NAS type.

Att,

Nataniel Klug