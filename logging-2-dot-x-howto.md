The "log" section of the radiusd.conf file is where the primary logging configuration for the FreeRADIUS server is located.

    log {
    	destination = files
    	file = ${logdir}/radius.log
    #	requests = ${logdir}/radiusd-%{%{Virtual-Server}:-DEFAULT}-%Y%m%d.log
    	syslog_facility = daemon
    	stripped_names = no
    	auth = no
    	auth_badpass = no
    	auth_goodpass = no
    #	msg_goodpass = ""
    #	msg_badpass = ""
    }


Log Destination
---------------

Destination for log messages.  This can be one of the following values:

* files  - log to "file", as defined below.
* syslog - send log messages to syslog (see also the "syslog_facility" below).
* stdout - log to standard output.
* stderr - log to standard error.

Note that the command-line debugging option "-X" overrides this option, and forces all logging to go to stdout.

**Default:**

    destination = files


Log File Location
-----------------

If the destination == "files", then the logging messages for the server are appended to the tail of this file.  Again, note that if the server is running in debugging mode, this file is NOT used.

**Default:**

    file = ${logdir}/radius.log


Requests Log
------------

If this configuration parameter is set, then log messages for a *request* go to this file.  This is a log file per request, once the server has accepted the request as being from a valid client.  Messages that are not associated with a request still go to radius.log defined above.

Note that not all log messages in the server core have been updated to use this new internal API.  As a result, some messages will still go to radius.log.  Patches are welcome to fix this behavior.

The file name is expanded dynamically.  You should ONLY user server-side attributes for the filename, i.e. things you control.  Using this feature MAY also slow down the server substantially, especially if you do things like SQL calls as part of the expansion of the filename.

The name of the log file should use attributes that don't change over the lifetime of a request, such as User-Name,
Virtual-Server or Packet-Src-IP-Address.  Otherwise, the log messages will be distributed over multiple files.

**Default (disabled):**

    requests = ${logdir}/radiusd-%{%{Virtual-Server}:-DEFAULT}-%Y%m%d.log


Syslog Facility
---------------

This option determines which syslog facility to use, if destination == "syslog"  The exact values permitted here are OS-dependent.  You probably don't want to change this.

**Default:**

    syslog_facility = daemon


Log User-Name Attribute
-----------------------

Log the full User-Name attribute, as it was found in the request.  The allowed values are: {no, yes}

**Default:**

    stripped_names = no


Log Authentication Requests
---------------------------

Log authentication requests to the log file.  The allowed values are: {no, yes}

**Default:**

    auth = no


Log Passwords
-------------

Log passwords with the authentication requests.

* auth_badpass  - logs password if it's rejected
* auth_goodpass - logs password if it's correct

The allowed values are: {no, yes}

**Default:**

    auth_badpass = no
    auth_goodpass = no


Log Additional Text
-------------------

Log additional text at the end of the "Login OK" messages.  For these to work, the "auth" and "auth_goodpass" or "auth_badpass" configurations above have to be set to "yes".

The strings below are dynamically expanded, which means that you can put anything you want in them.  However, note that this expansion can be slow, and can negatively impact server performance.

**Default (disabled):**

    msg_goodpass = ""
    msg_badpass = ""


Log Additional Debug Information
--------------------------------

Logging can also be enabled for an individual request by a special dynamic expansion macro:  **%{debug: #}**, where # is the debug level for this request (1, 2, 3, etc.).  For example:

    ...
    update control {
           Tmp-String-0 = "%{debug:1}"
    }
    ...

The attribute that the value is assigned to is unimportant, and should be a "throw-away" attribute with no side effects.

