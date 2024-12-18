# Operating Systems
## Officially supported
The following platforms are officially supported by the FreeRADIUS project, if FreeRADIUS fails to build or compile correctly of these platforms we consider this a bug, and you should [report it](http://bugs.freeradius.org).

If you are not constrained in your choice of operating systems, for maximum performance and reliability the FreeRADIUS core team recommend [FreeBSD](http://www.freebsd.org/).

* [FreeBSD](http://www.freebsd.org/)
* [Linux](http://www.kernel.org/)
  * [CentOS](http://www.centos.org/)
  * [Red Hat](http://www.redhat.com/)
  * [Debian](http://www.debian.org/)
  * [Ubuntu](http://www.ubuntu.com/)
* [macOS](http://www.apple.com/)
* [Solaris](http://www.oracle.com/technetwork/server-storage/solaris11/overview/index.html)

## Reported to work by the community
FreeRADIUS is reported to run on the following Operating Systems:
* [AIX](http://www.ibm.com/aix)
* [Cygwin](http://www.cygwin.com/)
* [FreeBSD](http://www.freebsd.org/)
* [HP-UX](http://www.hp.com/products1/unix/)
* [Linux](http://www.kernel.org/)
  * [CentOS](http://www.centos.org/)
  * [Debian](http://www.debian.org/)
  * [Open Mandriva](https://www.openmandriva.org/)
  * [Red Hat](http://www.redhat.com/)
  * [SUSE](http://www.opensuse.org/)
  * [Turbolinux](http://www.turbolinux.com/)
  * [Ubuntu](http://www.ubuntu.com/)
* [macOS](http://www.apple.com/)
* [NetBSD](http://www.netbsd.org/)
* [OpenBSD](http://www.openbsd.org/)
* OSF/Unix
* [Solaris](http://www.oracle.com/technetwork/server-storage/solaris11/overview/index.html)

Porting to other unix-like platforms should be easy. Due to the limited resources of the FreeRADIUS development team, we are not able to test each version on all platforms before release.

# Hardware Platforms

FreeRADIUS is reported to run on the following Hardware Architectures:
* IA-64 (Itanium & Itanium2)
* PPC (IBM POWER & PowerPC)
* Sparc
* Sparc64
* x86
* x86_64 (AMD64 & EMT64)

# Compilers
Our continuous test suite runs daily with:

* GCC >= 4.9
* clang >= 3.0

The server _should_ build with any fully C11 compliant compiler, if it doesn't, we consider that a bug, and you should [report it](http://bugs.freeradius.org).