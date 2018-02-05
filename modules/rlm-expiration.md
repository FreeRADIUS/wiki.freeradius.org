# rlm_expiration

## Synopsis

The expiration module implements support for the Expiration attribute.

[Default configuration](https://github.com/FreeRADIUS/freeradius-server/blob/v3.0.x/raddb/mods-available/expiration)

## Processing Sections
### authorize

When listed in the authorize section, the expiration module enforces the Expiration attribute. When the user’s account has a limited range of validity, the Session-Timeout attribute is updated to reflect this limited range.

If the Session-Timeout attribute already exists, the expiration module may decrease the value, but will never increase the value, of this attribute.

The format of the Expiration attribute is a date, as printed out by the date utility.

Example date
`$ date
Thu  3 Apr 2014 13:19:52 EDT`

_**Return codes**_

`userlock` The user’s account has expired.

`noop` No `control:Expiration` attribute was found.

`ok` A `control:Expiration` attribute was found, and the user’s account is still active.

## post-auth

Operates identically to the authorize section.

_Available after version 3.0.4_

## Expansions

_None._

## Directives

_None._