# Building radsniff
Currently only the version of radsniff in the master (3.1.x) branch supports statistics, which means you'll need to build the server from source.

Static builds do work on the master branch, meaning you can build radsniff as a standalone binary and not have issues with libfreeradius conflicts if you're running a stable version of the FreeRADIUS server on the same host.

## Dependencies
radsniff depends on _libpcap_ and _libcollectclient_. If _libpcap_ isn't available then radsniff will not be built. If _libcollectdclient_ isn't available radsniff will be build but without support for writing statistics out to a collectd socket.

On Debian/Ubuntu systems you can use apt to pull in the required packages:
```bash
sudo apt-get install libpcap-dev libcollectdclient-dev
```

There may be packages available for other systems, else you'll need to build the libraries from source, satisfying their dependencies manually.

## Configuring
If you wish to build a standalone binary of radsniff, you need to pass ``--with-shared-libs=no`` to the configure script, this disabled building with share libraries.

Note: It's also helpful to specify ``--prefix=DIR`` where DIR is your existing FreeRADIUS installation directory. This means radsniff will pick up the dictionaries automatically.

```bash
./configure --with-shared=no
```

The key lines of output you need to look for are:
```bash
checking for pcap_open_live in -lpcap... yes
checking for pcap_fopen_offline in -lpcap... yes
checking for pcap_dump_fopen in -lpcap... yes
checking for lcc_connect in -lcollectdclient... yes
...
checking for pcap.h... yes
checking for collectd/client.h... yes
```

If you've installed either of the libraries in non-standard locations, you can specify the various paths with ``--with-pcap-include-dir=DIR``, ``--with-pcap-lib-dir=DIR``, ``--with-collectdclient-include-dir=DIR`` and ``--with-collectdclient-lib-dir=DIR``.

## Building
The new build framework in 3.x has sufficient levels of awesome, to autocreate make targets for all the binaries automagically, meaning you can just:
```bash
make radsniff
```
and it'll build libfreeradius, the radsniff binary, and not the entire server.

Unfortunately there's no specific install target for radsniff, so for now it's best to copy the binary directly from ``./build/bin/radsniff`` to wherever you want it ``/usr/local/bin/`` for example...

## Running
3.1.x radsniff now has two main modes of operation, packet decode and output, and statistics generation. By default radsniff will decode packets, but not output any statistics.

There are three arguments we need to pass to radsniff to get it send stats to collectd:
* ``-W`` To turn on statistics the ``-W <period>`` flag is used. For integrating with collectd ``<period>`` should match your collectd interval (which is by default 10 seconds).
* ``-O`` To direct stats to a collectd instance ``-O <server>``, is used ``<server>`` can be an IP address, FQDN, or UNIX Socket. For this guide we'll need be using the collectd sock ``/var/run/collectdsock``.

