# Introduction

xlat expansions refer to the alternation and interpolation format used in all versions of the server.  xlat expansions the server to dynamically construct strings from the values of attributes and the output of various xlat C functions.  These xlat C functions allow data to be retrieved from various sources, or transformed (escaped, unescaped, encoded, decoded etc...).

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

In v3 output is either to a pre-allocated buffer (the length of which was determined at time of registration), or alternatively, the xlat function can allocate its own output buffers.  Unfortunately in both these cases the xlat evaluation code wouldn't necessarily treat the output buffers in a binary safe way, so xlat functions could not output binary data, this has lead to horrible hacks like xlat expansions creating attributes directly.

When xlat functions finish they return the number of bytes they wrote to the output buffer, or a negative integer if an error occurred. Unfortunately this change was introduced quite late in the v3.0.x branch and the xlat evaluation code was never updated to take advantage of it. 

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



