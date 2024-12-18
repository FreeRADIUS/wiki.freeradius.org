# List of xlat expansions

Also see the related list of [run-time variables](/config/run_time_variables).

## Provided by the server core

       name | description
------------|------------
attr        | attribute name of an attribute reference
attr_num    | attribute number of an attribute reference
debug_attr  | print to debug output all instances of current attribute, or all attributes in a list; expands to a zero-length string
debug       | dynamically change the debug level for the current request
hex         | convert to hex
integer     | convert to integer
length      | size of the given attributes in bytes
map         | processes data as a map string and applies to the current request
module      | returns current module processing the request
regex       | return named subcapture value from previous regex
string      | convert data to a string if possible
strlen      | length of given string `"%{strlen:Hello}" == 5`
tag         | tag of an attribute reference
vendor_num  | vendor number of an attribute reference
vendor      | vendor of an attribute reference
xlat        | xlat expand given string value


## Provided by modules

The following xlats are provided by particular modules, so the module will need
to be instantiated before the xlat will be available to use.

xlat names marked with an asterisk (*) take the name of the module
instantiation, so may change from that listed here if different instantiations
are used.


### rlm_cache

       name | description
------------|------------
cache*      | retrieve single attribute values from the cache


### rlm_date

       name | description
------------|------------
date*       | convert dates between different formats


### rlm_exec

       name | description
------------|------------
exec*       | execute external program


### rlm_expr

       name | description
------------|------------
base64      | encode string as base64: `"%{base64:foo}" == "Zm9v"`
base64tohex | convert base64 to hex: `"%{base64tohex:Zm9v}" == "666f6f"`
escape      | escape string similar to rlm_sql 'safe_characters': `"%{escape:<img>foo.jpg</img>}" == "=60img=62foo.jpg=60/img=62" `
expr*       | 
explode     | split an attribute into multiple new attributes based on a delimiter: `"%{explode:&ref <delim>}"`
hmacmd5     | generate HMAC-MD5 of string: `"%{hmacmd5:foo bar}" == "31b6db9e5eb4addb42f1a6ca07367adc"`
hmacsha1    | generate HMAC-SHA1 of string: `"%{hmacsha1:foo bar}" == "85d155c55ed286a300bd1cf124de08d87e914f3a"`
lpad        | left-pad a string: if `User-Name` is `"foo"`: `"%{lpad:&User-Name 6 x}" == "xxxfoo"`
md5         | get md5sum hash: `"%{md5:foo}" == "acbd18db4cc2f85cedef654fccc4a4d8"`
nexttime    | calculate number of seconds until next n hour(s), day(s), week(s), year(s), e.g. if it were 16:18, `%{nexttime:1h}` would expand to `2520`
pairs       | serialize attributes as comma-delimited string: `"%{pairs:request:}" == "User-Name = 'foo', User-Password = 'bar', ..."`
rand        | get random number from 0 to n-1 `"%{rand:10}" == "9"`
randstr     | get random string built from character classes (see below)
rpad        | right-pad a string: if `User-Name` is `"foo"`: `"%{rpad:&User-Name 5 -}" == "foo--"`
sha1        | get sha1 hash: `"%{sha1:foo}" == "0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33"`
sha256      | get sha256 hash: `"%{sha256:foo}" == "2c26b46b68ffc68ff99b453c1d30413413422d706..."`
sha512      | get sha512 hash: `"%{sha512:foo}" == "f7fbba6e0636f890e56fbbf3283e524c6fa3204ae29838..."`
tolower     | convert to lowercase: `"%{tolower:Bar}" == "bar"`
toupper     | convert to uppercase: `"%{toupper:Foo}" == "FOO"`
unescape    | reverse of escape: `"%{unescape:=60img=62foo.jpg=60/img=62}" == "<img>foo.jpg</img>"`
urlquote    | quote special characters in URI: `"%{urlquote:http://example.org/}" == "http%3A%47%47example.org%47"`
urlunquote  | unquote URL special characters: `"%{urlunquote:http%%3A%%47%%47example.org%%47}" == "http://example.org/"`

Characters that can be used in `randstr` are:

character | class
----------|------
        c | lowercase letters
        C | uppercase letters
        n | numbers
        a | alphanumeric
        ! | punctuation
        . | alphanumeric + punctuation
        s | alphanumeric + "./"
        o | characters suitable for OTP (easily confused removed)
        h | binary data as lowercase hex
        H | binary data as uppercase hex

Examples:

    "%{randstr:CCCC!!cccnnn}" == "IPFL>{saf874"
    "%{randstr:oooooooo}" == "rfVzyA4y"
    "%{randstr:hhhh}" == "68d60de3"


### rlm_idn

       name | description
------------|------------
idn*        | convert idn to ascii


### rlm_ldap

       name | description
------------|------------
ldap*       | do an LDAP query
ldapquote   | safely quote string for use in ldap query


### rlm_mschap

       name | description
------------|------------
mschap*     | extract ms-chap data from the request, e.g. `"%{mschap:User-Name}"` or `"%{mschap:Challenge}"`


### rlm_perl

       name | description
------------|------------
perl*       | call perl xlat function defined in `func_xlat`


### rlm_redis

       name | description
------------|------------
redis*      | run a redis query: `"%{redis:GET mykey}"`


### rlm_rest

       name | description
------------|------------
rest*       | retrieve text data from a URL
jsonquote   | quote data for use in JSON


### rlm_soh

       name | description
------------|------------
soh*        | translate SoH data, currently just `"%{soh:OS}"`


### rlm_sql

       name | description
------------|------------
sql*        | execute an SQL query, e.g. `"%{sql:SELECT user FROM users WHERE field = '%{Attribute-Name}';}%"


### rlm_unbound

         name | description
--------------|------------
unbound*-a    | lookup an A record in the DNS
unbound*-aaaa | lookup an AAAA record in the DNS
unbound*-ptr  | lookup a PTR record in the DNS


### rlm_unpack

       name | description
------------|------------
unpack      | unpack attribute data, e.g. `"%{unpack:&Class 0 integer}"` expands 4 octets at position 0 as an integer


### rlm_yubikey

       name | description
------------|------------
modhextohex | convert Yubikey modhex to standard hex, e.g. `"%{modhextohex:vvrbuctetdhc}" == "ffc1e0d3d260"`


## Other xlat expansions

These are provided by proto_dhcp:

        name | description
-------------|------------
client       | provides per-client config options i.e. %{client:ipaddr} or %{client:mymadeupoption}
dhcp         |
dhcp_options |

