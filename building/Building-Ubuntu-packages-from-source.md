## 2 Ways of installing FreeRadius on Ubuntu 

* Installing from repositories 
* Installing from source 

### Installing from repositories :

This is usually the easiest solution, but at the moment of writing (2016-06) both Ubuntu 16.04 and Ubuntu 14.04.4 contain packages which are EOL and which are not the latest in their own main version. So building from source is recommended as it will contain the latest version, which probably got most of the bugs sorted.

To Install :

Get the latest info about packages from Ubuntu :

      sudo apt-get update 

Then make sure the system is fully upgraded before installing freeradius :

      sudo apt-get upgrade

or

      sudo apt-get dist-upgrade

After that has been done we can use _apt_ to install freeradius :

     sudo apt-get install freeradius 

This will install the _common_,_ utils_, _ssl-cert_, _libdbi-perl_ and _libfreeradius2_ packages. As you probably going to connect to a database or use perl, you should probably also install (some of) the suggested packages. Such as: _freeradius-ldap_, _freeradius-postgresql_ and _freeradius-mysql_.

     sudo apt-get install freeradius-ldap        

The 'apt install freeradius' command will list all the suggested packages.

If all went well freeradius should now be installed and started. You can start or stop the server with :

     sudo service freeradius stop

or

     sudo service freeradius start

Suggested is to stop the service and until all is working use freeradius in debug mode.

      sudo freeradius -X

Right now your config files are in  :

      cd /etc/freeradius

Documentation and files for creation of certificates are in : 

      cd /usr/share/doc/freeradius


### Installing from source :

Installing from source can be daunting for people who never did it but as long as you read the output of the building process, it should tell you what went wrong or what is missing.

First step is to get the source. 2 places that currently offer the material :
FTP from freeradius.org via : [http://freeradius.org/download.html](http://freeradius.org/download.html)
HTTP download from GitHub : [https://github.com/FreeRADIUS/freeradius-server](https://github.com/FreeRADIUS/freeradius-server)
On GitHub select the branch you wish to install and press _clone or download_.

Make sure unzip or any other utility that can extract the zip is installed. If not :

     sudo apt-get install unzip

Here I download and extract the source to a temporary directory: (I'm using here a directory in my home folder.) Make sure the filename is the same as your downloaded one.

     cd /home/myusername/
     mkdir freeradius
     cd freeradius
     wget https://github.com/FreeRADIUS/freeradius-server/archive/v3.0.x.zip

     unzip v3.0.x.zip

or (depending on type of archive)

      tar zxf freeradius-server-W.X.Y.tar.gz

Now we need to go into the directory containing the source.

      cd freeradius-server-W.X.Y/

And run the following :

      fakeroot dpkg-buildpackage -b -uc

It will probably error out on a clean system. Because the build tools are not installed or because not all dependencies are installed.

Make sure fakeroot and build tools are installed (read the error message!!):

     sudo apt-get install fakeroot dpkg-dev quilt debhelper

In the file _debian/rules_ we might need to make some changes depending on other packages we might not have installed or support that we do (not) need. In one of my own installs, there was no iodbc database support required as I'm using ldap as a database backend. If I build it will give an error and fail to build because of that. You can modify your install in the following way to prevent the error.

     pico debian/rules

And just before 

     --without-rlm_eap_ikev2 \

I create a new line :

     --without-rml_sql_iodbc \

You can remove and add any module that you (do not) require this way.

It might also error with more unmet dependencies to be able to build. When you run the command to build the package but it errors out with a dependencies/conflict abortion, install them as well.

In my clean Ubuntu 16.04 install that meant :

     sudo apt-get install libcurl4-openssl-dev libcap-dev libgdbm-dev libiodbc2-dev libjson0-dev libkrb5-dev libldap2-dev libpam0g-dev libpcap-dev libperl-dev libmysqlclient-dev libpq-dev libreadline-dev libsasl2-dev libsqlite3-dev libssl-dev libtalloc-dev libwbclient-dev libyubikey-dev libykclient-dev libmemcached-dev libhiredis-dev python-dev samba-dev

After that we try again to build.

     fakeroot dpkg-buildpackage -b -uc	 

If it errors out after it has already started building the deb files, you are sometimes better off starting anew. If that happens :

     cd /home/myusername/freeradius
     rm -R freeradius-server-W.X.Y

And unpack the archive again. I.e.

     unzip v3.0.x.zip

And edit the debian/rules commenting out or adding depending on the error it gave you at the end of the build.

After build has completed without any errors we can finally install.

     cd /home/myusername/freeradius
     sudo dpkg -i *freeradius*_W.X.Y*_*.deb

The install might show errors. Read the error !! Ask questions on freeradius list if you cannot figure it out. v2 will fail install often on open_ssl issues. Quick thing to change to prevent just that error is to edit a config file so freeradius will not complain about ssl that might be vulnerable. ( /etc/freeradius/eap.conf (v2) or /etc/freeradius/modules-enabled/eap )
