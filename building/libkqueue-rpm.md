
Latest version of libkqueue can be found [here](https://github.com/mheily/libkqueue/releases).

```bash
# Replace v2.1.0 with latest version
VERSION=2.1.0
wget https://github.com/mheily/libkqueue/archive/v${VERSION}.tar.gz
tar -xvzf v${VERSION}.tar.gz
cd ./libkqueue-${VERSION}
sed -ie "s/Version:.*/Version:    ${VERSION}/" ./libkqueue.spec
sed -ie "s/%{_mandir}\/man2\/kevent.2.*//" ./libkqueue.spec
sed -ie "s/^%{make_install}.*/%{make_install} \&\& rm -rf %{_libdir}\/*.a \&\& rm -rf %{_libdir}\/*.la/" ./libkqueue.spec
autoreconf -i
cd ..
tar -czf ./libkqueue-${VERSION}.tar.gz ./libkqueue-${VERSION}

# Only required if you don't already have a build environment setup
mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
cp ./libkqueue-${VERSION}.tar.gz ~/rpmbuild/SOURCES/libkqueue-${VERSION}.tar.gz
rpmbuild -ba ./libkqueue-${VERSION}/libkqueue.spec
```