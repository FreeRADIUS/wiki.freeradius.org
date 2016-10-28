Do *one* of the following:

* From the git repository (version 3.0.x)

```bash
git clone https://github.com/FreeRADIUS/freeradius-server.git
cd freeradius-server
git checkout v3.0.x
```

* From the git repository (version 3.0.12 - latest released version)

```bash
git clone https://github.com/FreeRADIUS/freeradius-server.git
cd freeradius-server
git checkout release_3_0_12
```

On Debian and Ubuntu only: a small error slipped into the release of 3.0.12 so the packages don't build cleanly, so then do:

```
git cherry-pick e5cce05
```

* From a zip file (version 3.0.x - most recent stable version, though unreleased)

```bash
wget https://github.com/FreeRADIUS/freeradius-server/archive/v3.0.x.zip
unzip v3.0.x.zip
cd freeradius-server-3.0.x/
```

* From a zip file

```bash
wget https://github.com/FreeRADIUS/freeradius-server/archive/v3.0.12.zip
unzip v3.0.12.zip
cd freeradius-server-3.0.12/
```

(On Debian and Ubuntu, use 3.0.11 instead due to a minor packaging bug, or use the git instructions above.)

