# Introduction

xlat expansions refer to the alternation and interpolation format used in all versions of the server.  xlat expansions the server to dynamically construct strings from the values of attributes and the output of various xlat C functions.  These xlat C functions allow data to be retrieved from various sources or transformed (escaped, unescaped, encoded, decoded etc...).

When used in the server configuration xlat expansions usually look like this ``%{My-Attribute}``  or in the case of functions ``%{my_func:%{My-Attribute}}``.  xlat expansions can be nested to any depth.

You should not need to alter the xlat evaluation or parsing code itself as the xlat framework is extensible via runtime registered xlat functions.

## How xlat functions work in v3

In v3 the base64 encoding function looked like this:

```c
static ssize_t base64_xlat(UNUSED TALLOC_CTX *ctx, char **out, size_t outlen,
                           UNUSED void const *mod_inst, UNUSED void const *xlat_inst,
                           REQUEST *request, char const *fmt)
{
    size_t        inlen;
    uint8_t        *p;
    TALLOC_CTX    *tmp_ctx = NULL;
    ssize_t        ret;

    VALUE_FROM_FMT(tmp_ctx, p, inlen, request, fmt);

    /*
     *  We can accurately calculate the length of the output string
     *  if it's larger than outlen, the output would be useless so abort.
     */
    if ((FR_BASE64_ENC_LENGTH(inlen) + 1) > outlen) {
        REDEBUG("xlat failed");

        talloc_free(tmp_ctx);

        return -1;
    }

    ret = fr_base64_encode(*out, outlen, p, inlen);
    talloc_free(tmp_ctx);

    return ret;
}
```

## Instances and instantiation

The xlat functions are passed the instance data of the module that registered them or NULL or the xlat function was registered internally.

There is no per-use instantiation.  The `xlat_inst` argument was envisaged to be used for this purpose, but the necessary changes were never made to the xlat framework, it is currently unused by all xlat functions.

## Input

When the xlat function is called it is passed the result of any nested expansions.  The results of these nested expansions are concatenated together in the `fmt` argument.  This is a normal C buffer and is not binary safe.

Any nested xlat expansions that involve dynamic expansions i.e. reading the value of an attribute or calling another xlat function, will be passed through the escaping function that was registered at the same time as the xlat function when it was passed to `xlat_register()`.  If no escaping function was registered, the nested values will be provided to the xlat function untouched.

There are two primary limitations of this method.

1. It is not possible to determine which fmt components were derived from expansions and which ones were provided as part of the configuration as literal strings.
2. As the fmt string is not binary safe, any binary values must be passed in as attribute references `%{base64:&Attr-To-Encode}` which are then expanded by the xlat function itself.  That's what the `VALUE_FROM_FMT(tmp_ctx, p, inlen, request, fmt);` macro does in the above `base64` code.

### Output

In v3 output is either to a pre-allocated buffer (the length of which was determined at the time of registration) or alternatively, the xlat function can allocate its own output buffers.  Unfortunately, in both these cases, the xlat evaluation code wouldn't necessarily treat the output buffers in a binary safe way, so xlat functions could not output binary data, this has lead to horrible hacks like xlat expansions creating attributes directly.

When xlat functions finish they return the number of bytes they wrote to the output buffer, or a negative integer if an error occurred. Unfortunately, this change was introduced quite late in the v3.0.x branch and the xlat evaluation code was never updated to take advantage of it. 

## How xlat functions work in v4

```c
static xlat_action_t xlat_base64(TALLOC_CTX *ctx, fr_cursor_t *out,
                 REQUEST *request, UNUSED void const *xlat_inst, UNUSED void *xlat_thread_inst,
                 fr_value_box_t **in)
{
    size_t        alen;
    ssize_t        elen;
    char        *buff;
    fr_value_box_t    *vb;
    /*
     *    If there's no input, there's no output
     */
    if (!in) return XLAT_ACTION_DONE;

    if (fr_value_box_list_concat(ctx, *in, in, FR_TYPE_OCTETS, true) < 0) {
        RPEDEBUG("Failed concatenating input");
        return XLAT_ACTION_FAIL;
    }

    MEM(vb = fr_value_box_alloc_null(ctx));
    alen = FR_BASE64_ENC_LENGTH((*in)->vb_length);
    MEM(buff = talloc_array(ctx, char, alen + 1));

    elen = fr_base64_encode(buff, alen + 1, (*in)->vb_octets, (*in)->vb_length);
    if (elen < 0) {
        RPEDEBUG("Base64 encoding failed");
        talloc_free(vb);
        return XLAT_ACTION_FAIL;
    }

    rad_assert(elen <= alen);

    if (fr_value_box_strsnteal(vb, vb, NULL, &buff, elen, false) < 0) {
        RPEDEBUG("Failed assigning encoded data buffer to box");
        talloc_free(vb);
        return XLAT_ACTION_FAIL;
    }

    fr_cursor_append(out, vb);

    return XLAT_ACTION_DONE;
}
```

## Instances and Instantiation

In FreeRADIUS <= 3 xlat functions were created and registered primarily by modules so it made sense to pass module instance data to the xlats.  This has changed in v4 and module instance data is not provided directly to xlat expansions.  If module instance data is still required it can be passed to the `xlat_register_async()` function and stored by the per-use instantiation function in per-use instance data.

At registration time per-use global, and per-use thread-specific instantiation functions can be provided, these are used to create per-use instance data, which can be used by xlat functions to pre-compile literal components of input data, or store handles to things like stored procedures.

For the majority of internal xlat functions the per-use xlat instance data is not used and can be marked as `UNUSED`.

## Input

v4 xlat functions get a linked list of boxed values (`fr_value_box_t`) as input.  These value boxes are basically a C union and a bit of additional data like taint state, and length (for variable length buffers).  They're a convenient way of emulating dynamic typing.  The individual chunks of input data passed into the xlat function are *not* concatenated by the calling code.  This is pretty useful for a few reasons.

- The taint status (whether it came from a trusted source) of the different input values is maintained.  The new style xlat functions can emulate the escaping behaviour of the old ones by calling an escape function on any tainted input boxes.
- You can emulate arguments by looking for boxes containing only whitespace.  More work needs to be done on this, but it is functional today.
- Non-String/buffer types don't automatically get smashed into their string representations, meaning the code can be more efficient.

Because many functions operate with a single argument there's a function provided to concatenate all the input arguments to a string if required.  It does this in place for efficiency reasons, and the xlat evaluation code does not expect input arguments to be in a sane/usable state once the xlat function has returned. See the call to `fr_value_box_list_concat` in the above base64 code for an example.

After concatenation you can access the input data as a single string with `(*in)->vb_strvalue` and get the length with `(*in)->vb_length`. You must only use binary safe functions to operate on these strings as they could contain embedded `\0` bytes.

If an xlat function wants to process the individual discreet chunks of input it can either traverse each box's `->next` pointer manually or use a `fr_cursor_t` and the associated functions.

```
fr_cursor_t cursor;
fr_value_box_t *vb;

vb = fr_cursor_init(&cursor, in);
if (!vb) {
	// Missing first argument
}

vb = fr_cursor_next(&cursor);
if (!vb) {
	// Missing second argument
}
```
_An example of using a cursor to traverse input values_

Note: The discreet boxes don't actually map to arguments.  This is likely to change in future where literals will be broken out into separate boxes on whitespace by the xlat evaluation code.

## Output
In v4 all xlat functions must allocate their own output memory.  A talloc context `ctx` is provided for this.

Function output should be zero or more `fr_value_box_t`.  A cursor `out` is provided to insert additional values into.  When the xlat function is called the cursor will be at the end of any previously output.  You can use the full range of functions on this cursor, but it's recommended not to advance it, and to `fr_cursor_append(&cursor, <vb>)` any values your function produces.  If xlat function errors during its execution you can then call `fr_cursor_list_free(&cursor)` to free any values that have already been added to the output cursor so execution is idempotent.

See `src/lib/util/value.c` and `src/lib/util/cursor.c` for value box and cursor manpulation functions and documentation.

xlat functions should return `XLAT_ACTION_DONE` on success or `XLAT_ACTION_FAIL` on failure.  Exactly what happens on failure depends on where the expansion was called.  For example, if an xlat expansion is called as part of an update block and it fails, then the entire update block is rolled back.



