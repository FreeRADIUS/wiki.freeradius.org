# Async xlat in v4

This needs doing.

We will create an `unlang_xlat_t` in the unlang compiler and interpreter.

The xlat code needs to be updated to have a function which does one level of xlat expansion.  i.e. a function which takes an `xlat_t`, does something, and returns an `xlat_t`.  The idea is that the existing code can loop over calling this function:

    get xlat_node
    do {
        xlat_node = xlat_expand(xlat_node, data...)
    } while (xlat_node)

Right now, `xlat_aprint()` calls itself recursively.  This is really the main function that needs fixing.

There should then be an `unlang_xlat()` function in `src/main/unlang_interpret.c`.  It should interpret the xlat nodes.  It gets called when a module needs an xlat expanded.

We need another new function, `unlang_xlat()`.  This function will return YIELD if necessary.  A module will call this function in order to do an xlat expansion.

The `unlang_xlat()` function will take an xlat node, and call `xlat_expand() iteratively.  The `unlang_xlat_t` data structure on the stack will need to contain both the current xlat node, and a linked list of `value_box_t`, which are the various portions of the data being expanded.  Note that xlat expansions can be recursive, e.g. `%{sql:SELECT %{User-Name}...}`.  We probably do not need a recursive use of `unlang_xlat()`, as the recursive expansions are just getting more data, as in `User-Name = bob`.  The output of the recursive expansion is just a series of `value_box_t`.  Note that this means every call to `xlat_expand()` may push multiple values onto the `value_box_t` list, and `unlang_xlat()` has to take this into account.

The `unlang_xlat()` function will keep calling `xlat_expand()` until there are no more xlat nodes to run. At that point, `unlang_xlat()` will coalesce all of the data, and return it to the caller in a form that the caller needs, typically a string, or `value_box_t`