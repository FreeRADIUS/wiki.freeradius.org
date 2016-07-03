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

