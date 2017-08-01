Simplicity
==========

Even with the features and complexity of the server. it is designed to
be simple to [install](/radiusd/INSTALL) and
[configure](http://wiki.freeradius.org). When installing from source,
just follow the instructions on the [download](/building/Home)
page. Or, install a [package](OS) for your operating system.

Once the server has been installed, edit the `raddb/users` file, and add
the following line of text at the top:

:   ``` {.config}
    bob   Cleartext-Password := "hello"
    ```

Then in a terminal window, start the server in debugging mode:

:   ``` {.config}
    $ radiusd -X
    ```

When the debug messages finish scrolling, go to another terminal window
on the same machine, and type:

:   ``` {.config}
    $ radtest bob hello localhost 0 testing123
    ```

The `radtest` program will send a test packet to the server, and the
server will respond. The server *should* respond with an `Access-Accept`
message. If it doesn't, something has gone wrong, and the debugging
output of the server will explain any problems.

If it does work (and it should), then almost all of the available
authentication types will now work. In Version 2.0 with the test
certificates, most EAP authentication types will also work, including
PEAP and EAP-TTLS.

The default configuration has been designed to work in the widest
possible set of circumstances. Only minor edits to the configuration
files are required to enable most features. Those edits are usually
limited to un-commenting a feature everywhere it is mentioned in the
configuration files (e.g. use of LDAP). Then, adding a small amount of
per-site configuration (e.g. point to a local LDAP server).

In general, making large changes to the default configuration files will
break common use-cases. We *strongly* recommend using a careful process
for editing the configuration files, such as the following:

****
1.  Save a backup of the current configuration.
2.  Make a minor edit.
3.  Test it.
4.  If it works, repeat from the top.

Of course, not all changes may work. If the changes do not behave as
expected, then the configuration can be reverted to the known-working
backup. The server will then continue to run with the last
configuration, giving you time to understand more about how it should be
configured to meet your needs.
