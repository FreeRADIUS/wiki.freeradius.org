IP Pools
========

IP addresses can be allocated from [IP
pools](http://wiki.freeradius.org/modules/Rlm_sqlippool). The
pools can be managed in SQL [databases](Database), or via local
GDBM files.

When using an SQL database, the IP pools have been tested with
pools containing tens of millions of addresses.
[Scalability](Scalability) is not a problem.
