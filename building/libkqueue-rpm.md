Latest version of libkqueue can be found [here](https://github.com/mheily/libkqueue/releases).

```bash
# Replace v2.1.0 with latest version
VERSION=2.3.1
wget https://github.com/mheily/libkqueue/archive/v${VERSION}.tar.gz
tar -xvzf v${VERSION}.tar.gz
cd ./libkqueue-${VERSION}
cmake3 -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=lib .
make
cpack3 -G RPM
yum install *.rpm
```