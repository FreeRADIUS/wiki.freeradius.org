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

