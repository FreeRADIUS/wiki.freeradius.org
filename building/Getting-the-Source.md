Do *one* of the following:

* From the git repository (version 3.0.x)

```bash
git clone https://github.com/FreeRADIUS/freeradius-server.git
cd freeradius-server
git checkout v3.0.x
```

* From the git repository (version 3.0.12 - latest released version)

Note - a small error slipped into the release of 3.0.12 so the packages don't build cleanly, hence the cherry-pick

```bash
git clone https://github.com/FreeRADIUS/freeradius-server.git
cd freeradius-server
git checkout release_3_0_12
git cherry-pick e5cce05
```

* From a zip file (version 3.0.x - most recent stable version, though unreleased)

```bash
wget https://github.com/FreeRADIUS/freeradius-server/archive/v3.0.x.zip
unzip v3.0.x.zip
cd freeradius-server-3.0.x/
```

* From a zip file (version 3.0.11 - does not work with version 3.0.12 due to a minor packaging bug)

```bash
wget https://github.com/FreeRADIUS/freeradius-server/archive/v3.0.11.zip
unzip v3.0.11.zip
cd freeradius-server-3.0.11/
```
