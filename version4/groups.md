# Groups

Diameter has an attribute type "group".  Unlike TLVs, it contains
other top-level attributes.  There are policy-based limits on what the
group can contain, but the wire format has no such restrictions.

We need groups in RADIUS for attributes of type [[tag]], or TLVs.

The difference with RADIUS is that there are wire-format restrictions
on which attributes can go where.  It's probably safe to just ignore
those restrictions in the server core, which simplifies it.

The encoder can just complain if it finds something where the VP
parentage doesn't match the DA parentage.

