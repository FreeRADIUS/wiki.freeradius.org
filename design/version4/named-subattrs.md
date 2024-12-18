# Named Subattributes

We need a way to name sub-attributes, as used with [[groups]].

The simple thing is to leverage the OID syntax:

    Foo.bar.baz = 5

Refers to `baz` which MUST be a sub-TLV (or in a group) of `bar`,
which is itself a sub-TLV (or in a group) of `Foo`.

This syntax needs to be compatible with the `users` file, `sql`, etc.

The proposal is to leverage the OID syntax again, this time by allowing partial OIDs:

    Foo.bar.baz = 5
    .stuff = 6

Here `stuff` refers to the *previously used attribute* `foo.bar.baz`.
The `.stuff` says it's in the same parentage as `Foo.bar.baz`,
i.e. `foo.bar`.  If you want to go up a parentage, you could do
`..stuff`, which would say it's parented from `foo`.

