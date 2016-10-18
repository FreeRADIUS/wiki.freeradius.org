# Building on RHEL7 or Centos7
## Upgrading GCC

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

Or can set ``CC=/opt/rh/devtoolset-3/root/usr/bin/gcc`` in your environment, which works just as well.

If you're building on older versions of RedHat then you'll need to compile GCC from source.

## Building from source
### Dependencies

```bash
yum -y install libtalloc-devel
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