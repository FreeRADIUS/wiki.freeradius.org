For debugging purposes it's sometimes useful to have a version of libldap built with debugging symbols.

Here's how to do that on macOS.

## Configure OpenLDAP
```
CFLAGS='-g3' LDFLAGS='-L/usr/local/openssl/lib' CPPFLAGS='-I/usr/local/openssl/include' ./configure --prefix=/usr/local/openldap/ --enable-debug --enable-hdb=no --enable-bdb=no --with-tls=openssl
```

Note - You must build against OpenSSL else you'll get obscure crashes, or the server will error out during startup as non of the TLS options will work.

## Build OpenLDAP
```
LTCFLAGS='-g3'
export LTCFLAGS
make depend -j8
make -j8
sudo make install STRIP=''
```

## Configure/build FreeRADIUS
```
./configure --with-libfreeradius-ldap-lib-dir=/usr/local/openldap/lib --with-libfreeradius-ldap-include-dir=/usr/local/openldap/include
make
make install
```