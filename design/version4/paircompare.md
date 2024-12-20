# Async paircompare()

`paircompare()` functionality returns a dynamic comparison for attributes.  It's like taking a dynamic value every time the attribute is referenced, except that the comparison is done in the `paircompare()` function.

The underlying `paircompare()` function compares attributes to attributes in the request.  It doesn't handle lists.  It's called from:

* `cond_eval.c` - compare one attribute in an `if` statement.  The list is handled by the `cond_eval` functionality.
* `rlm_files.c` - users file comparisons
* `rlm_preprocess.c`, hints and huntgroups comparisons
* `rlm_sql.c` - compare `radcheck` and `radgroupcheck` items retrieved from SQL

It's also referenced from:

* `unlang_compile.c` comment as mark up paircompare fixups (?)

Pair comparisons are registered by:

* `src/main/pair.c` - `paircompare_register_byname()`
* `rlm_expiration` - the `Expiration` attribute. *can be replaced by more unlang`
* `rlm_expr` - `Prefix`, `Suffix`, `Connect-Rate`, `Packet-Type`, and `Response-Packet-Type`, src/dst ip/port, virtual server, packet processing stage, *Most of these can either be deleted, or replaced with dynamic xlats.*
* `rlm_ldap` - `LDAP-Group`
* `rlm_logintime` - `Current-Time`, and `Time-of-Day`, *can either be deleted, or replaced with dynamic xlats.*
* `rlm_sql` - `SQL-Group`
* `rlm_sqlcounter` - counter thing?
* `rlm_test`
* `rlm_unix`- `Unix-Group` *should probably be replaced with a map?*
* `rlm_winbind` `Winbind-Group`

## Suggestions

Many of these can be replaced by dynamic xlats (e.g. `Current-Time`)

The various group functionalities could be replaced (badly) with xlat expansions: `%{ldap-group:%{User-Name} sales}` which is shit, but would work.

It would be ideal to allow `LDAP-Group == sales` to still work.  But that means fixing all of the callers of `paircompare()` to allow for it to be async, too.  That's a lot of work.  The better approach is just to replace comparison of virtual attributes with xlat expansions, *or* function calls.

It's likely easier to fix the callers so that they call `map` functions, and then just get rid of the `paircompare()` functionality altogether.

## Proposal

Note: `paircompare()` is part condition, and part map.  i.e. the `paircompare()` functions *set* some attributes unconditionally, and return true/false for comparison of other attributes.  It also takes a list of `VALUE_PAIR`s, and thus is not really amenable to converting it to maps and conditions.

* get rid of as many virtual attributes as possible first, e.g. `Prefix`.  People don't really use them
* convert most of the rest to xlat expansions, and document that in `raddb/README`
* get rid of `hints` and `huntgroups`.  Sorry.
* move the sql `radcheck` and `radgroupcheck` functions to return conditions and maps?
* update `PAIR_LIST` to return conditions and maps?, and `pairlist_read()`, which is only called from `rlm_files` and `rlm_preprocess`
* add callbacks to templates?  i.e. virtual attributes like `paircompare()`, with a duplicate API.

