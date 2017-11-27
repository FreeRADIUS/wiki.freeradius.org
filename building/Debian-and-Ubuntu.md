# Building on Debian or Ubuntu

Building Debian packages (including Ubuntu) of FreeRADIUS from source is kept as simple as possible.

## Building the stable release (v3.0)

At the time of writing (October 2016), the Debian and Ubuntu repositories contain only version 2 of the server, which is end-of-life and no longer officially supported by the FreeRADIUS project. You are therefore recommended to following the instructions here and build your own packages.

Building packages should be very simple. First obtain a copy of the source and unpack it. Second, build the packages.

### Getting the source

[[include:Getting-the-Source]]

### Building Packages

Having retrieved whichever version of the source you require, build the FreeRADIUS packages:

```bash
sudo apt-get install devscripts quilt debhelper fakeroot equivs
fakeroot debian/rules clean
sudo mk-build-deps -ir debian/control
dpkg-buildpackage -rfakeroot -b -uc
```

This will build packages in the parent directory, which can be installed with ``dpkg -i``.

### Building from source

Alternatively, rather than building packages, you can build the source directly. Note that you will need to ensure all required dependencies are installed first (such as `libssl-dev`).

```bash
# Use ./configure --enable-developer if you're debugging issues, or using unstable code.
./configure
make
sudo make install
```


## Building development versions (v4.0)

Note that version 4 is for developers only. **Do not use these versions unless you know what you are doing.**

### Upgrading GCC

Older versions of Debian and Ubuntu use GCC < 4.8, which lacks support for the C11 features needed to build FreeRADIUS >= v4.0.x.

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

### Installing hard dependencies

```bash
sudo apt-get install libssl-dev libtalloc-dev libkqueue-dev
```

### Building

Get the source as described above, then:

```bash
./configure --enable-developer
make
sudo make install
```


