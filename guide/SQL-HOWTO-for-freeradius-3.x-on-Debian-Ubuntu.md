FreeRADIUS Server works out of the box with a [[rlm_sql|large list of SQL servers]]

These instructions are based on the [[SQL HOWTO]] which is outdated for the 3.x versions and will describe the configuration on Debian/Ubuntu (tested with FreeRADIUS 3.0.11 version).

##Before You Start##

Before starting with FreeRADIUS, please make sure your server is up and configured on your network, that you have your [[SQL]] server of choice (MySQL, PostgreSQL etc) installed and running, and that your [[NAS]] is configured to send [[RADIUS]] requests to your RADIUS server.

We have some sample configs for [[Cisco]] [[NAS]] [[Cisco|available here]].

##Getting Started##

Firstly, you need to install FreeRADIUS Server on your system. As the premiere open source RADIUS suite it is included as a standard package with numerous Operating Systems and has [[packages]] for many others. Installation is most easily accomplished by installing a binary package (rpm, deb), but if you have a less well known operating system you may need to [[build]] your own.

##Basic configuration##

See [[Basic configuration HOWTO]]

##Setting up the RADIUS database##

First, you should create a new empty 'radius' database in SQL and create the schema for the database. There is an SQL script file for each SQL type in `mods-config/sql/main/*sql_dialect*/schema.sql` (or where you installed FreeRADIUS to).

Next create a database user with permissions to that database. You could of course call the database and the user anything you like but you probably should stick with 'radius' for the user to keep things simple. You can find sql statements to achieve this in `mods-config/sql/main/*sql_dialect*/setup.sql`


##Example setup of MySQL database##
The following section show how this can be done when using a MySQL database.

###Create MySQL Database###

    mysql -uroot -p
      CREATE DATABASE radius;
      exit

###Create SQL schema###
 Execute the following command to create the database schema

    mysql -uroot -p radius < mods-config/sql/main/mysql/schema.sql


###Create MySQL User and grant permissions###
 In the file `mods-config/sql/main/mysql/setup.sql` set a more secure password than 'radpass'. If your SQL server is running on a different machine you also have to replace the `localhost` with your radius server.

    mysql -uroot -p radius < `mods-config/sql/main/mysql/setup.sql`

##Configuring FreeRADIUS to use SQL##

Edit /etc/mods-available/sql module and enter the SQL dialect, driver, server, username and password details to connect to your SQL server and the RADIUS database. The database and table names should be left at the defaults if you used the default schema. For testing/debug purposes, uncomment the `logfile = ...` line - FreeRADIUS will dump all SQL commands to the log file specified here.

Next enable the sql module by executing

<pre>
    cd mods-enabled
    ln -s ../mods-available/sql sql
</pre>

Edit /`sites-available/default` (or whatever site config you use) and uncomment the line containing `sql` in the `authorize{}` section. 

Additionally, edit `sites-available/inner-tunnel` and uncomment the line containing 'sql' under "authorize {}".

Also uncomment the line saying 'sql' in the accounting{} section to tell FreeRADIUS to store accounting records in SQL as well.

Optionally add or uncomment 'sql' to the session{} section if you want to do Simultaneous-Use detection.

Optionally add or uncomment 'sql' to the post-auth{} section if you want to log all Authentication attempts to SQL.

Optionally, if you want to strip all realm names (i.e. you want user joe@domain.com to authenticate as just 'joe'), then in file `mods-config/sql/main/*sql_dialect*/queries.conf` , under the 'query config: username' section, you MAY need to adjust the line(s) referring to sql_user_name. For example, in uncomment the line:
 
    sql_user_name = '%{Stripped-User-Name}'

...and comment out the following line referring to just User-Name. If you want to see what's happening here, switch on all the logging options in radiusd.conf and run radiusd in debug mode (-X) to see what's happening : you'll see " user@domain" being passed to SQL when using User-Name, but just "user" when using Stripped-User-Name. Of course, set all your other SQL options as needed (database login details, etc)

'''You should not change/delete any other lines in the config file without reading and understanding the comments!'''

The config you use (e.g. sites-enabled/default) should then look something like this:
 <pre>
 authorize {
        preprocess
        chap
        mschap
        suffix
        eap
        # We leave "files" enabled to allow creation of test users in the "users" file
        files
        sql
        pap
 }
 accounting {
        # We leave "detail" enabled to _additionally_ log accounting to /var/log/radius/radacct
        detail
        sql
 }
</pre>
 
##Populating SQL##

Now we create some dummy data in the database to test against. 

To check whether FreeRADIUS now properly works with the SQL database, we create a user entry in radcheck.

<pre>
mysql -u radius -p
>  use radius;
>  insert into radcheck (username,attribute,op,value) values("fredf", "Cleartext-Password", ":=", "wilma");
</pre>

Then try authenticating as the user with radtest. If you get Accept-Accept response, you can continue with group setup if you wish.

To configure groups, we'll have to do these steps:
* In usergroup, put entries matching a user account name to a group name.
* In radcheck, put an entry for each user account name with a 'Cleartext-Password' attribute with a value of their password.
* In radreply, create entries for each user-specific radius reply attribute against their username
* In radgroupreply, create attributes to be returned to all group members

Here's a dump of some example 'radius' tables from a MySQL database (With PostgreSQL the formatting will look slightly different but it uses exactly the same content).

This example includes three users, one with a dynamically assigned IP by the NAS (fredf), one assigned a static IP (barney), and one representing a dial-up routed connection (dialrouter):

      mysql> select * from radusergroup;
      +----+---------------+-----------+
      | id | UserName      | GroupName |
      +----+---------------+-----------+
      |  1 | fredf         | dynamic   |
      |  2 | barney        | static    |
      |  2 | dialrouter    | netdial   |
      +----+---------------+-----------+
      3 rows in set (0.01 sec)
 
      mysql> select * from radcheck;
      +----+----------------+--------------------+------------------+------+
      | id | UserName       | Attribute          | Value            | Op   | 
      +----+----------------+--------------------+------------------+------+
      |  1 | fredf          | Cleartext-Password | wilma            | :=   |
      |  2 | barney         | Cleartext-Password | betty            | :=   |
      |  2 | dialrouter     | Cleartext-Password | dialup           | :=   |
      +----+----------------+--------------------+------------------+------+
      3 rows in set (0.01 sec)
 
      mysql> select * from radreply;
 
      +----+------------+-------------------+---------------------------------+------+
      | id | UserName   | Attribute         | Value                           | Op   |
      +----+------------+-------------------+---------------------------------+------+
      |  1 | barney     | Framed-IP-Address | 1.2.3.4                         | :=   |
      |  2 | dialrouter | Framed-IP-Address | 2.3.4.1                         | :=   |
      |  3 | dialrouter | Framed-IP-Netmask | 255.255.255.255                 | :=   |
      |  4 | dialrouter | Framed-Routing    | Broadcast-Listen                | :=   |
      |  5 | dialrouter | Framed-Route      | 2.3.4.0 255.255.255.248         | :=   |
      |  6 | dialrouter | Idle-Timeout      | 900                             | :=   |
      +----+------------+-------------------+---------------------------------+------+
      6 rows in set (0.01 sec)
 
      mysql> select * from radgroupreply;
      +----+-----------+--------------------+---------------------+------+
      | id | GroupName | Attribute          | Value               | Op   |
      +----+-----------+--------------------+---------------------+------+
      | 34 | dynamic   | Framed-Compression | Van-Jacobsen-TCP-IP | :=   |
      | 33 | dynamic   | Framed-Protocol    | PPP                 | :=   |
      | 32 | dynamic   | Service-Type       | Framed-User         | :=   |
      | 35 | dynamic   | Framed-MTU         | 1500                | :=   |
      | 37 | static    | Framed-Protocol    | PPP                 | :=   |
      | 38 | static    | Service-Type       | Framed-User         | :=   |
      | 39 | static    | Framed-Compression | Van-Jacobsen-TCP-IP | :=   |
      | 41 | netdial   | Service-Type       | Framed-User         | :=   |
      | 42 | netdial   | Framed-Protocol    | PPP                 | :=   |
      +----+-----------+--------------------+---------------------+------+
      12 rows in set (0.01 sec)
 

In this example, 'barney' (who is a single user dialup) only needs an attribute for IP address in radreply so he gets his static IP - he does not need any other attributes here as all the others get picked up from the 'static' group entries in radgroupreply.

'fred' needs no entries in radreply as he is dynamically assigned an IP via the NAS - so he'll just get the 'dynamic' group entries from radgroupreply ONLY.

'dialrouter' is a dial-up router, so as well as needing a static IP it needs route and mask attributes (etc) to be returned. Hence the additional entries.

'dialrouter' also has an idle-timeout attribute so the router gets kicked if it's not doing anything - you could add this for other users too if you wanted to. Of course, if you feel like or need to add any other attributes, that's kind of up to you!

Note the operator ('op') values used in the various tables. The password check attribute MUST use :=. Most return attributes should have a := operator, although if you're returning multiple attributes of the same type (e.g. multiple Cisco- AVpair's) you should use the += operator instead otherwise only the first one will be returned. Read the docs for more details on [[Operators|operators]].

If you're stripping all domain name elements from usernames via realms, remember NOT to include the domain name elements in the usernames you put in the SQL tables - they should get stripped BEFORE the database is checked, so name@domain will NEVER match if you're realm stripping (assuming you follow point 2 above) – you should just have 'name' as a user in the database. Once it's working without, and if you want more complex realm handling, go back to work out not stripping (and keeping name@domain in the db) if you really want to.

##Test##

Fire up radiusd again in debug mode (radiusd -X). The debug output should show it connecting to the SQL database. Use radtest (or NTradPing) to test again - the user should authenticate and the debug output should show FreeRADIUS talking to SQL.

Congratulations. You're done!

##Additional Snippets##

* To use encrypted passwords in radcheck use the attribute 'Crypt-Password', instead of 'Cleartext-Password', and just put the encrypted password in the value field. ( i.e. UNIX crypt'd password). 
* To get NTradPing to send test accounting (e.g. stop) packets it needs arguments, namely acct-session-time. Put something like 'Acct-Session-Time=99999' into the 'Additional RADIUS Attributes' box when sending stops.
* If you have a [[Cisco]] nas, set the cisco-vsa-hack

* Running a backup FreeRADIUS server and need to replicate the RADIUS database to it? You can follow Colin Bloch's basic instructions at http://www.ls-l.net/mysql/ and got replication setup between two MySQL servers. Real easy. Read the MySQL docs on replication for more details.

On the subject of backup servers. If you want to run TWO MySQL servers and have FreeRADIUS fall over between them, you'll need to do something like this: 
* duplicate your `mods-enabled/sql` file twice
* change the module names from `sql { ....` to something like `sql sql1 { ....` and `sql sql2 { ....`
* edit the second copy to reflect connecting to your backup server  
* in the different sections of your site config (eg. sites-enabled/default) change the 'sql' entry to a 'redundant sql' one, like this:
 <pre>
    redundant sql {
      sql1 
      sql2 
    }
 </pre>

Note that if FreeRADIUS fails over to the second MySQL server and tries to update the accounting table (radacct), nasty things might possibly happen to your replication setup and database integrity as the first MySQL server won't have got the updates...

##Troubleshooting##
If you think FreeRADIUS doesn't correctly process the SQL authentication, accounting, etc. enable the query logging by uncommenting the `logfile = ...` line in the sql module and make sure that the user the radius server at has write permissions on that file. After restarting the radius server the SQL queries should be written to that file. If it still isn't working use radiusd -X to look for error messages.

##See also##

* [[rlm_sql]]
* [[rlm_sqlcounter]]
* [[rlm_sqlippool]]
* [[SQL HOWTO]]