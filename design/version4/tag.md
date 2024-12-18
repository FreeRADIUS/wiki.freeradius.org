# Tags in v4

Tagged attributes are a horrible hack that breaks all kinds of things
and makes the internal API more complex.  e.g. `TAG_ANY`.  The
solution appears to be to use re-use [[groups]].

We would create 32 attributes, `Tag-0` though `Tag-31`.  These
attributes would be of type `group`, as with Diameter.

The tagged attributes would then be put into the appropriate group by
the parser.  e.g.

    Tunnel-Type:1 := 3

would end up as

    Tag-1 {
       Tunnel-Type := 3
    }

We could also leverage [[named-subattrs]], and do:

    Tag-1.Tunnel-Type := 3

The encoder could then see that these attributes were of type `tag`,
(or `group`, with a tag option), and then call a special encoding
routine.

The TLV stack used by the RADIUS encoder would remain the same, as it
enforces DA / TLV parentage.  Using groups would break this, as the VP
parentage would be different from the DA / TLV parentage.

The solution to that problem is for the `tag` encoder to just create
it's own TLV stack, and call the normal encoder.  Since tagged
attributes can only be of type `string` or `integer`, the TLV stack
will only ever be one entry deep.

The encoder can write the tag, and then call the `encode_value()`
function to do the actual encoding.  Or, just do it itself, as it's
all pretty simple.

