# Building FreeRADIUS
## Supported Platforms

FreeRADIUS is known to run on a large number of 32 and 64bit [platforms](building/Platforms). In general the build procedure between platforms is very similar, the main differences are how to satisfy dependencies, and how to [build packages](building/Packages).

## FreeRADIUS hard dependencies

See [platform specific build instructions](#building-from-source) for how to satisfy these dependencies on your platform.

### libtalloc (since >= v3.0.x)

[Talloc](https://talloc.samba.org/talloc/doc/html/index.html) is a hierachical memory allocator used/created by the Samba project.

It is used **heavily** in version >= v3.0.x and greatly simplifies managing complex trees of memory allocation, and local slab allocation.

### C11 (since >= v3.1.x)

Since v3.1.x  FreeRADIUS has a hard dependency on C11 support (available in GCC >= 4.9.0).

If you see an error message like

> configure: error: FreeRADIUS requires support for the C11 _Generic keyword

That means your compiler does not support C11, and you'll need to one that does.

For clang this means versions >= 3.0 (released after 2011-12-01), and GCC versions >= 4.9 (released after 2014-04-22).

### libkqueue or native kqueue support (since >= v4.0.x)

kqueue is the eventing interface used by the BSDs (including OSX).  After evaluating the native eventing APIs of different operating systems and wrappers such as libuv, libev, libevent[2] etc... the FreeRADIUS core team decided to standardise on kqueue.

For Linux users, this means there's a hard dependency on them shim library, libkqueue, which wraps epoll (the native Linux eventing API), providing a kqueue compatible interface.

## Getting the source

[[include:Getting-the-Source]]

## Building from Source

```bash
tar -zxvf freeradius-<version>.tar.gz	 
./configure	 
make	  
sudo make install	 
```

Don't forget to read supplied documentation first, including the configuration files. Many configuration options are documented inline, in the configuration files themselves. 

Platform specific instructions are available for:

- [Debian and Ubuntu](Debian and Ubuntu)
- [macOS](macOS)
- [RHEL and Centos](RHEL and Centos)
- [Solaris](Solaris)
- [Suse](Suse)

## Building Packages

The FreeRADIUS source contains build rules for several different types of system packages. If your operating system has a packaging system (dpkg, rpm, tgz), it is usually easier to install the appropriate packages instead of directly installing from source. However this may not always be the recommended approach as many systems seem to lag behind with very old versions of FreeRADIUS. In that case it may be better to build packages from source.

Platform specific instructions are available for:

- [Debian and Ubuntu](Debian and Ubuntu#building-packages)
- [RHEL and Centos](RHEL and Centos#building-packages)
- [Suse](Suse#building-packages)

# See Also
* [Pre-Built Packages](pre-built-packages)