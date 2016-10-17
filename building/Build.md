# Building FreeRADIUS
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

On OSX, FreeBSD, OpenBSD etc no additional dependencies are required.

## Supported Platforms

FreeRADIUS is known to run on a large number of 32 and 64bit [platforms](building/Platforms), however building on some of them may require following special procedures here.

## Building from Source

If your operating system has support for a package manager such as .deb or .rpm file format, we recommend that you follow the instructions in the [next section](Build#building-packages) instead:

```bash
tar zxvf freeradius-<version>.tar.gz	 
./configure	 
make	  
sudo make install	 
```

Don't forget to read supplied documentation first, including the configuration files. As with many free software projects, FreeRADIUS could use more documentation. Until such documentation is available, the only place that configuration items are documented is in the configuration files themselves.	 
	 
If you have problems when trying to run FreeRADIUS, and you see error messages like:

> rlm_sql: Could not link driver rlm_sql_mysql: file not found	 

Then the shared libraries on your system are misconfigured.

## Building on Debian or Ubuntu
### Dependencies

```bash
sudo apt-get install libtalloc-dev libkqueue-dev
```

### Building/Installing
```bash
# Use ./configure --enable-developer if you're debugging issues, or using unstable code.
./configure
make
sudo make install
```

## Building on RHEL7 or Centos7
### Upgrading GCC

RHE7 ships with GCC 4.8.5 but we require GCC >= 4.9.0 for FreeRADIUS >= v3.1.x.

Fortunately the ``devtoolset-3`` series of packages provides a later version of GCC.

Follow the instructions here to enable the [devtoolset-3 repository](https://www.softwarecollections.org/en/scls/rhscl/devtoolset-3/).

To install:

```bash
yum -y install devtoolset-3-gcc devtoolset-3-gcc-c++
```

and then to get to a shell with the correct environment:

```bash
scl enable devtoolset-3 bash
```

### Dependencies

```bash
yum -y install libtalloc-dev
```

#### libkqueue
Latest version can be found [here](https://github.com/mheily/libkqueue/releases).

```bash
# Replace v2.1.0 with latest version
VERSION=2.1.0
wget https://github.com/mheily/libkqueue/archive/v${VERSION}.tar.gz
tar -xvzf v${VERSION}.tar.gz
cd ./libkqueue-${VERSION}
sed -ie "s/Version:.*/Version:    ${VERSION}/" ./libkqueue.spec
sed -ie "s/%{_mandir}\/man2\/kevent.2.*//" ./libkqueue.spec
sed -ie "s/^%{make_install}.*/%{make_install} rm -rf \${RPM_BUILD_ROOT}\/%{_libdir}\/*.a \&\& rm -rf \${RPM_BUILD_ROOT}\/%{_libdir}\/*.la/" ./libkqueue.spec
autoreconf -i
cd ..
tar -czf ./libkqueue-${VERSION}.tar.gz ./libkqueue-${VERSION}

# Only required if you don't already have a build environment setup
mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
cp ./libkqueue-${VERSION}.tar.gz ~/rpmbuild/SOURCES/libkqueue-${VERSION}.tar.gz
rpmbuild -ba ./libkqueue-${VERSION}/libkqueue.spec
```

### Building/Installing

```bash
# Use ./configure --enable-developer if you're debugging issues, or using unstable code.
./configure
make
sudo make install
```

Or can set ``CC=/opt/rh/devtoolset-3/root/usr/bin/gcc`` in your environment, which works just as well.

If you're building on older versions of RedHat then you'll need to compile GCC from source.

## Building on macOS

If you don't have homebrew package manager installed, [do it now](http://brew.sh)... it'll make your life on macOS far simpler.

```bash
brew install talloc
# Use ./configure --enable-developer if you're debugging issues, or using unstable code.
./configure
make
sudo make install
```

## Building on Solaris

Please see: [Solaris](building/Solaris)

# Building Packages
The FreeRADIUS source contains build rules for several different types of system packages. If your operating system has a packaging system (dpkg, rpm, tgz), it is usually easier to install the appropriate packages instead of directly installing from source. However this may not always be the recommended approach as many systems seem to lag behind with very old versions of FreeRADIUS. In that case it may be better to build packages from source.

## Debian

Building Debian packages of FreeRADIUS from source is kept as simple as possible. Please refer to the [Debian package](Debian) page for full instructions.

The above page also includes instructions on building with Oracle support or installing Debian **backports** packages for older systems.

## Building Ubuntu packages

If you're using Ubuntu, you should first check whether your desired version of FreeRADIUS is available in the Ubuntu package repositories, because that will save you the trouble of building packages yourself. As of March 2016, the Ubuntu repositories contain only version 2 of the server, which is end-of-life. Please see: [[http://packages.ubuntu.com/freeradius]].

For build instructions, please follow the instructions (building Ubuntu Packages)[building/Building-Ubuntu-packages-from-source] or follow the same directions as [building Debian packages](Debian#building-debian-packages) on the main Debian page.

## Building RedHat packages

Please refer to the information on the Red Hat specific page (Red Hat FAQ)[guide/Red Hat FAQ].

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