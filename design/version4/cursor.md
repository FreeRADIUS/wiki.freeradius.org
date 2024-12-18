# Cursors in unlang

The current code doesn't allow for cursors as is done in the C source.  For example:

    if (Attribute-Name == "foo") {

is an *implicit* loop over all attributes of name `Attribute-Name`,
and it stops when it finds one having value `foo`.

Even worse, once that check has been done, all information about it
has been lost.  Any further `update` sections have to start from
scratch again.

This problem gets worse with the introduction of [[groups]].

The proposed solution is to add a `cursor` syntax, ala:

    cursor CONDITION {
        ...
    }

The `CONDITION` is (mostly) just a condition as is used in `if`
statements.  The difference here is that it should be limited to
`attribute OP value`, and not allow string conditions, etc.

The `cursor` code will loop over the list specified in the condition,
`request:User-Name`, and look for matching attributes.  When found, it
will run the inner block.

The default list for the inner block will then be *not* the `request`,
or even the list specified by the condition.  Instead, the default
list will be the *attribute*, if the attribute is of type `group`.

Or...if the attribute is not of type `group`, we need a special
`cursor` attribute (ala `foreach`), which contains a *reference* to
the given attribute.

## Possible extension - expressions

Implementing path expressions in a similar style to jpath and xpath may also be useful.

i.e.

	&foo[5].bar[<10]

Would select all 'bar' sub-tlvs of the 5th instance of attribute 'foo' with a value less than 10 (i'm sure there's a better syntax).

Much of the infrastructure to do this is already there, it'd just require extensions of the templates to allow a linked list of attribute references, enhancement of the attribute selector syntax, and adding evaluation code in the tmpl_cursor* functions.

In the case of if conditions, the code already deals with multiple values for the LHS or RHS OK, so no modifications would need to be made to the condition evaluator.

One good use case for the expression syntax is being able to apply updates to a subset of grouped TLVs.

    cursor &[*].Tunnel-Medium-Type[==802] {
	    update .. {
	    	Tunnel-Type := VLAN
	    }
    }

Here the '..' means change the ctx to the parent of the current attribute.

### Comments...

Selectors / xpath expressions have their pros and cons.  Pro: no conditional syntax, so no need to forbid conditions which don't make sense.  con: any xpath-style syntax is going to be very, very, complex.

we could start out just by allowing conditions `(Foo == bar)`, and limiting the parser to require that the LHS is an attribute.  It's a bit of a hack, but easy to explain.

For complex paths, it's probably easier for the poor administrator to just write nested cursors.

    cursor (&foo[5]) {
        cursor (.bar < 10) {
            ...
        }
    }