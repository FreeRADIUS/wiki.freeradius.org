# Building on macOS
## Hard dependencies
If you don't have homebrew package manager installed, [do it now](http://brew.sh)... it'll make your life on macOS far simpler.

```bash
brew install talloc
```

## Getting the source

[[include:Getting-the-Source]]

## Building from source

```bash
# Use ./configure --enable-developer if you're debugging issues, or using unstable code.
./configure
make
sudo make install
```