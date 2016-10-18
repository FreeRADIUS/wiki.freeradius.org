# Building on Debian or Ubuntu
## Upgrading GCC
Older versions of Debian and Ubuntu use GCC < 4.8, which lacks support for the C11 features needed to build FreeRADIUS >= v3.1.x.

In order to switch to GCC 4.9
```bash
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install g++-4.9

# Then select GCC 4.9
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 100 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 50 --slave /usr/bin/g++ g++ /usr/bin/g++-4.9 
sudo update-alternatives --config gcc

# Choose option 3 from the dialogue
```

## Building from source
### Dependencies

```bash
sudo apt-get install libtalloc-dev libkqueue-dev
```

Building Debian packages of FreeRADIUS from source is kept as simple as possible. Please refer to the [Debian package](Debian) page for full instructions.

The above page also includes instructions on building with Oracle support or installing Debian **backports** packages for older systems.

### Building/Installing

```bash
# Use ./configure --enable-developer if you're debugging issues, or using unstable code.
./configure
make
sudo make install
```

## Building Packages

If you're using Ubuntu, you should first check whether your desired version of FreeRADIUS is available in the Ubuntu package repositories, because that will save you the trouble of building packages yourself. As of March 2016, the Ubuntu repositories contain only version 2 of the server, which is end-of-life. Please see: http://packages.ubuntu.com/freeradius.

FreeRADIUS shipped with the current versions of Debian (both `Jessie` and `Wheezy`) are several years out of date. This means that they are no longer officially supported by the FreeRADIUS project.

Building Debian packages should be very simple. First obtain a copy of the source and unpack it. Second, build the packages.

### Getting the source

[[include:Getting-the-Source]]

### Building the packages

Having retrieved whichever version of the source you require, build the FreeRADIUS packages:

```bash
sudo apt-get install devscripts
fakeroot debian/rules clean
sudo mk-build-deps -ir debian/control
dpkg-buildpackage -rfakeroot -b -uc
```

This will build packages in the parent directory, which can be installed with ``dpkg -i``.