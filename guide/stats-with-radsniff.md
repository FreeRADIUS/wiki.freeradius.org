## Why?
On UDP buffer exhaustion FreeRADIUS has no idea that packets are being dropped as it's not informed, it only knows then it has dropped packets due to the packet queue being full. There's no way around this other than by having an external process snooping on the traffic.

radsniff should also work fine with other RADIUS servers. So where vendors have failed to provide sufficient instrumentation in their products, you can use radsniff to monitor their product's reliability and performance.

## What packet rate can radsniff deal with?
That is entirely dependent on your machine. But on a modern intel i7 processor it seems to be able to handle between 25k-30k PPS before libpcap starts reporting packet loss. This should be fine for the majority of installations, it's very unlikely that a real world FreeRADIUS installation (which will usually involve coupling FreeRADIUS with a directory or database component) would be able to handle 30k PPS anyway.

When libpcap starts reporting loss, radsniff will mute itself, that is stop writing stats out to the collectd socket, and stop writing stats to stdout. The time it's muted for will be double the timeout period.

## Building radsniff
Currently only the version of radsniff in the master (3.1.x) branch supports statistics, which means you'll need to build the server from source.

Static builds do work on the master branch, meaning you can build radsniff as a standalone binary and not have issues with libfreeradius conflicts if you're running a stable version of the FreeRADIUS server on the same host.

### Dependencies
radsniff depends on libpcap and libcollectclient. If libpcap isn't available then radsniff will not be built. If libcollectdclient isn't available radsniff will be build but without support for writing statistics out to a collectd socket.

On Debian/Ubuntu systems you can use apt to pull in the required packages:
```bash
sudo apt-get install libpcap-dev libcollectdclient-dev
```

There may be packages available for other systems, else you'll need to build the libraries from source, satisfying their dependencies manually.

### Configuring
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

### Building
The new build framework in 3.x has sufficient levels of awesome, to autocreate make targets for all the binaries automagically, meaning you can just:
```bash
make radsniff
```
and it'll just build libfreeradius, the radsniff binary, and not the entire server.

Unfortunately there's no specific install target for radsniff, so for now it's best to copy the binary directly from ``./build/bin/radsniff`` to wherever you want it ``/usr/local/bin/`` for example...

### Running
#### Manually
3.1.x radsniff has two main modes of operation, packet decode and output, and statistics generation. By default radsniff will decode packets, but not output any statistics.

There two arguments required to switch radsniff to stats mode and get it to write those stats to collect:
* ``-W`` - To turn on statistics the ``-W <period>`` flag is used. For integrating with collectd ``<period>`` should match your collectd interval (which is by default 10 seconds).
* ``-O`` - To direct stats to a collectd instance ``-O <server>``, is used. ``<server>`` can be an IP address, FQDN, or UNIX Socket. For this guide we'll need be using a UNIX socket ``/var/run/collectd-sock`` (the collectd default).

You may also (optionally) pass one or more ``-i <interface>`` arguments to specify interfaces to listen on. If no ``-i`` flags are passed radsniff will listen on all interfaces it can, opening separate PCAP sessions for each interface.

You may also (optionally) pass ``-P <pidfile>`` to make radsniff daemonize and write out a pid file. This is primarily for use in init scripts.

An example invocation for running radsniff as stats daemon would be something like:
```bash
radsniff -q -i eth0 -P /var/run/freeradius/radsniff.pid -W 10 -O /var/run/collectd-sock > /var/log/radsniff.log 2>&1
```

#### Via init script
Bundled in the ``scripts/`` directory of the freeradius tarball is ``radsniff.init``. This is intended for use on Debian systems.

By default the init script will pass the following arguments:
```bash
-P "$PIDFILE"-N "$PROG" -q -W 10 -O /var/run/collectd-unixsock
```

``-P`` and ``-N`` cannot be changed, but the other arguments can be overridden by creating ``/etc/default/radsniff`` and adding ``RADSNIFF_OPTIONS=<your options>``.

If you need to run multiple instances of radsniff, simply ``ln -s /etc/init.d/radsniff /etc/init.d/<instance name>``. ``PIDFILE`` will then be set to ``/var/run/freeradius/<instance name>.pid``, and the init script will load ``/etc/default/<instance name>``.

## collectd
To date I haven't managed to get radsniff to connect to collectd over UDP (keep getting connection refused errors, suspect libcollectdclient bug), but have been successful getting it work over unix socket.

Nothing special is required 

## munin
_But I don't use collectd, I use munin!_. Munin only provides an interface to pull stats from it's various plugins, this makes integrating it with radsniff more difficult (it would have to write stats out to a file which would then be consumed by a munin plugin).

Although both munin and collectd use rrdtool, there doesn't appear to be an easy way to read stats directly from RRD files (or create graphs from them directly). It seems like the simplest way of pulling the statistics across, is with a plugin wrapping rrdtool, which consolidates stats from the collectd RRD files, mangles the field names a bit, and writes them out in the format munin expects.

So [here's a plugin](https://raw.github.com/FreeRADIUS/freeradius-server/master/scripts/munin/radsniff) which does just that.

There are a few caveats when interpreting stats from it. Firstly the resolution is very different between collectd (10 seconds) and munin (300 seconds). This means the stats you see in munin are AVERAGED from 5mins worth of collectd stats.

If packet loss or retransmissions occur you may see a deceptively small spike on the munin graph. To 
