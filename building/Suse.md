
## Building SUSE packages

On SUSE Linux it should be a simple matter of taking the latest FreeRADIUS release tarball and dropping it in ``/usr/src/packages/SOURCES`` along with the other files from the``suse/`` directory inside the tarball with the exception of ``freeradius.spec`` which goes in ``/usr/src/packages/SPECS``

Then simply run:

```bash
rpmbuild -ba /usr/src/packages/SPECS/freeradius.spec
```

``rpmbuild`` will tell you if you are missing any build dependencies. If so, simply install them with ``yast2 -i packagename-devel`` then rerun ``rpmbuild``
