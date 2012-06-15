## Building/Installing
#### Standard Method
```bash
./configure
make
make install
```

This works on most Solaris systems.  You will need to have a compiler installed, and libraries for any services you intend to use. (OpenSSL, LDAP, Oracle, etc.)

### Method 1
These instructions should also work on Solaris 10 (x86_64) with minimal changes.

The source compiles perfectly once OS dependencies etc. are met. The main additional modules compiled here were ``rlm_ldap`` and ``rlm_sql``.

#### Solaris System Headers
Solaris 10 will likely require you to fix the system headers.

http://sunfreeware.com/indexsparc10.html

Do the following as root:
```bash
cd /usr/local/lib/gcc-lib/sparc-sun-solaris2.10/3.3.2/install-tools/
vi mkheaders.conf
```

Then put the line ``SHELL=/bin/sh`` on the first line of the ``mkheaders.conf`` file. It should look something like the following:
```bash
SHELL=/bin/sh
SYSTEM_HEADER_DIR="/usr/include"
OTHER_FIXINCLUDES_DIRS=""
FIXPROTO_DEFINES=""
STMP_FIXPROTO="stmp-fixproto"
STMP_FIXINC="stmp-fixinc"
```

Then run the following command as root. It may take several minutes to rebuild the headers.
```bash
./mkheaders
```

#### Solaris Packages
Solaris 10 has versions of Openssl and OpenLDAP installed, however they do not fullfill the compile requirements for freeradius functionality.
You should go to http://sunfreeware.com/ and get the packages there, and also resolve any unmet dependencies.

If you have other modules you are concerned with that are not building correctly, don't trust the OS packages.  Look for equiv packages and try the build with them installed as well.
```bash
download package
gunzip packagename.gz
sudo pkgadd -d packagename
```
#### Installing FreeRadius
Now you can use the standard, configure make sudo make install.

#### Runtime Environment
In order for the ldap queries to work, the following needs to be set as an environmental variable, OR if you're handy with compiler flags you can take care of it during the compile with the ``-RLIBDIR`` linker flag.

```bash
export LD_LIBRARY_PATH="/usr/local/lib/;/usr/local/freeradius/lib"
```

The two locations in the above path are for access to the ``libgcc_s.so.1`` libraries and the ``rlm_ldap`` libraries respectively.

### Method 2

Notes for building on Solaris. (SPARC or x86 shouldn't matter)

#### Specific info for this method
* Many packes are available from [Blastwave](http://www.blastwave.org) which installs everything into the base ``/opt/csw``.
* For this MySQL was built and installed in ``/usr/local``.
* Solaris uses a different runtime link loading method than linux (which uses ldconfig). For this reason, you either set -R (runtime flags) alongside -L flags during compilation and loading OR set ``LD_LIBRARY_PATH`` at runtime, which then defines a pathlike structure for loading libs at runtime. 
If you build most server software from source, -R is recommended if you want to know what, which and where stuff goes and which versions of of libraries are linked to.
_Note: Setting ``LD_LIBRARY_PATH`` negates and runtime paths already encoded in binaries._

#### Building
From the above the next few lines can be used to build freeradius on solaris (you can use this approach to build any software).

```bash
export PATH='/usr/sbin:/usr/bin:/opt/csw/bin:/opt/csw/gcc3/bin:/usr/ccs/bin:/opt/SUNWspro/bin'
export CFLAGS='-I/usr/local/openldap/include/ -I/usr/local/mysql/include/mysql/ -I/opt/csw/include/'
export LDFLAGS='-L/usr/local/openldap/lib/ -R/usr/local/openldap/lib -L/usr/local/mysql/lib -R/usr/local/mysql/lib -L/opt/csw/lib -R/opt/csw/lib'
export LD_OPTIONS='-L/usr/local/openldap/lib/ -R/usr/local/openldap/lib -L/usr/local/mysql/lib -R/usr/local/mysql/lib -L/opt/csw/lib -R/opt/csw/lib'
./configure --prefix=/usr/local/freeradius-1.1.2-mysql-ldap --with-ldap --with-mysql-dir=/usr/local/mysql-5.0.21
gmake
gmake install
```
## Running
SMF manifests and installation instructions for Solaris 10 can be found [here](https://github.com/alandekok/freeradius-server/tree/master/scripts/solaris).