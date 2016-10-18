# Building FreeRADIUS
## Supported Platforms

FreeRADIUS is known to run on a large number of 32 and 64bit [platforms](building/Platforms). In general the build procedure between platforms is very similar, the main differences are how to satisfy dependencies, and how to [build packages](building/Packages).

## FreeRADIUS hard dependencies
### libtalloc (since >= v3.0.x)

[Talloc](https://talloc.samba.org/talloc/doc/html/index.html) is a hierachical memory allocator used/created by the Samba project.

It is used **heavily** in version >= v3.0.x and greatly simplifies managing complex trees of memory allocation, and local slab allocation.

On Centos/Suse/RHEL install with
```bash
sudo yum install libtalloc-devel libtalloc
```

On Debian/Ubuntu install with
```bash
sudo apt-get install libtalloc-dev libtalloc
```

On OSX install with
```bash
brew install talloc
```

### C11 (since >= v3.1.x)

Since v3.1.x  FreeRADIUS has a hard dependency on C11 support (available in GCC >= 4.9.0).

If you see an error message like

> configure: error: FreeRADIUS requires support for the C11 _Generic keyword

That means your compiler does not support C11, and you'll need to one that does.

For clang this means versions >= 3.0 (released 2011-12-01), and GCC versions >= 4.9 (released 2014-04-22).

### libkqueue or native kqueue support (since >= v4.0.x)

kqueue is the eventing interface used by the BSDs (including OSX).  After evaluating the native eventing APIs of different operating systems and wrappers such as libuv, libev, libevent[2] etc... the FreeRADIUS core team decided to standardise on kqueue.

For Linux users, this means there's a hard dependency on them shim library, libkqueue, which wraps epoll (the native Linux eventing API), providing a kqueue compatible interface.

On Centos/Suse/RHEL see the build instructions here.

On Debian/Ubuntu install with
```bash
sudo apt-get install libkqueue-dev libkqueue
```

On macOS, FreeBSD, OpenBSD etc... no additional dependencies are required.

## Getting the source

[[include:Getting-the-Source]]

## Building from Source

```bash
tar zxvf freeradius-<version>.tar.gz	 
./configure	 
make	  
sudo make install	 
```

Don't forget to read supplied documentation first, including the configuration files. Many configuration options are documented inline, in the configuration files themselves. 

Platform specific instructions are available for:

- [macOS](macOS)
- [RHEL and Centos](RHEL and Centos)
- [Debian and Ubuntu](Debian and Ubuntu)
- [Solaris](Solaris)
- [Suse](Suse)

## Building Packages

The FreeRADIUS source contains build rules for several different types of system packages. If your operating system has a packaging system (dpkg, rpm, tgz), it is usually easier to install the appropriate packages instead of directly installing from source. However this may not always be the recommended approach as many systems seem to lag behind with very old versions of FreeRADIUS. In that case it may be better to build packages from source.

## Building SUSE packages

On SUSE Linux it should be a simple matter of taking the latest FreeRADIUS release tarball and dropping it in ``/usr/src/packages/SOURCES`` along with the other files from the``suse/`` directory inside the tarball with the exception of ``freeradius.spec`` which goes in ``/usr/src/packages/SPECS``

Then simply run:

```bash
rpmbuild -ba /usr/src/packages/SPECS/freeradius.spec
```

``rpmbuild`` will tell you if you are missing any build dependencies. If so, simply install them with ``yast2 -i packagename-devel`` then rerun ``rpmbuild``

## Building RPM packages with Oracle Support

If you wish to use Oracle you will need to recompile FreeRADIUS on a machine 
that has Oracle development libraries installed. FreeRADIUS is known to work both with a full Oracle installation as well as with the [Oracle Instant Client SDK](http://www.oracle.com/technology/tech/oci/instantclient/index.html). Once built the resulting RPM package can be deployed with just the [Oracle Instant Client](http://www.oracle.com/technology/tech/oci/instantclient/index.html) (No need for the SDK on production machines)

Most rpm packages available do not included oraclesql.conf due to the fact that they also don't contain the Oracle driver module (due to copyright restrictions).

If you have the Oracle header files in a sane location it should be a simple matter of taking the latest FreeRADIUS release tarball and 
dropping it in ``/usr/src/packages/SOURCES`` along with the other files from the ``suse/`` or ``redhat/`` directory inside the tarball with the exception of ``freeradius.spec`` which goes in ``/usr/src/packages/SPECS``

Then edit ``/usr/src/packages/SPECS/freeradius.spec`` and change:

```
%define _oracle_support 0
```

to:

```
%define _oracle_support 1
```

Then simply run:

```
rpmbuild -ba /usr/src/packages/SPECS/freeradius.spec
```
# See Also
* (Standard package)[building/packages]