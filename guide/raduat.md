# radauth.sh

Note: only ``radclient`` from version >= 3.0.7 will work correctly with ``raduat.sh``

## Overview
Bundled with the server source is a small shell script for writing test suites. It uses ``radclient`` to send test requests, and validate responses.

It currently supports PAP and CHAP, no EAP methods. Because of this raduat is only really useful for ISPs and carriers, as they generally use simple authentication methods, but need to perform end to end testing involving complex business logic.

## Creating a test request
A test request consists of ``<attribute><op><value>`` pairs, separated by newlines.

### Attribute
Every request must contain a ``Packet-Type=[Access-Request|Accounting-Request]`` pair, to set the type of test request.

In addition to ``Packet-Type`` it may contain any attribute (including VSAs) found in the FreeRADIUS dictionaries.

Commonly used attributes are:
- ``User-Name``
- ``User-Password``
- ``NAS-IP-Address``
- ``Acct-Session-Id``
- ``NAS-Port-ID``
- ``Acct-Session-Time``

### Op
Op should be one of the assignment operators:

- ``=``
- ``:=``
- ``+=``

### Value
Only static values are currently supported, though backtick (exec) expansion may be added in the future.

Only strings should be wrapped in single quotes, other values should be left unquoted.

For binary attributes the value may be specified as an unquoted string with a 0x prefix e.g. ``0xffffff``

### Tags
The following comment tags are supported. They should be placed at the top of the request file.

- ``# serial`` - Ensures this test is not executed in parallel with other tests

### Comments
Any line starting with ``#`` is treated as a comment.

### Example
An example test request might look like this:

```bash
# serial
# My test access request
Packet-Type=Access-Request
User-Name='test_user001@test.realm.example.org'
User-Password='testing123'
NAS-IP-Address=127.0.0.1
```

This test file would produce an ``Access-Request``, with the attributes ``User-Name``, ``User-Password`` and ``NAS-IP-Address``.

## Creating a response filter
A response filter consists of ``<attribute><op><value>`` pairs, separated by newlines.

Response filters verify that the response from the server matches what's expected.

The pairs in the response filter do not need to be in the same order as the response, but every attribute in the response must be matched by a line in the response filter.

### Attribute
Every response filter must contain a ``Response-Packet-Type=[Access-Accept|Access-Reject|Accounting-Response]`` pair, to set the type of response expected.

In addition to ``Response-Packet-Type`` it may contain any attribute (including VSAs) found in the FreeRADIUS dictionaries.

Commonly used attributes are:
- ``Reply-Message``
- ``User-Name``
- ``Class``
- ``Framed-IP-Address``
- ``Framed-Route``
- ``Session-Timeout``

### Op

Op should be one of the comparison operators:
- ``<`` - Less than
- ``>`` - Greater than
- ``<=`` - Less than or equal to
- ``>=`` - Greater than or equal to
- ``==`` - Equality
- ``!=`` - Inequality
- ``~=`` - Matches
- ``!~`` - Does not match
- ``*= ANY`` - Present
- ``!* ANY`` - Not present

### Value
Value format is the same as the request.

### Example
An example test response might look like this:
```bash
Response-Packet-Type==Access-Accept
Reply-Message=='Welcome to foocorp'
```

## Creating a test suite
By default ``raduat`` will look for a folder in the current working directory called ``tests``.

If it finds one, it will attempt to execute each of the test requests in lexicographical order of the file names.

``raduat`` isn't limited to a single directory, it will work perfectly happily with a directory hierarchy, in which case the tests are executed depth first then in lexicographical order.

Only files with names that match the format ``test[0-9]{3}.*`` will be processed. Each request file must also be paired with a response file. A response file has the same name as the request, with an ``_expected`` suffix.

For example the response file for ``test000_check_static_ip`` would be ``test000_check_static_ip_expected``.

### Example

```bash
mkdir -p ./tests/static_ip

echo "Packet-Type=Access-Request
User-Name=test_user001@test.realm
User-Password=testing123" >> ./tests/static_ip/test000_check_static_ip

echo "Response-Packet-Type==Access-Accept
Framed-IP-address=192.168.0.1" >> ./tests/static_ip/test000_check_static_ip_expected

echo "Packet-Type=Access-Request
User-Name=test_user002@test.realm
User-Password=testing123" >> ./tests/static_ip/test001_check_static_ip

echo "Response-Packet-Type==Access-Accept
Framed-IP-address=192.168.0.2" >> ./tests/static_ip/test001_check_static_ip_expected
```

