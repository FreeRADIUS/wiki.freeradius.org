# How to Implement Huntgroups in SQL with FreeRADIUS 2.x/3.x

## What are Huntgroups?
Huntgroups provide a mechanism to group NAS's into groups. Each NAS can be a member of a particular hunt group. When a user authentication request arrives you can then tag the request with the hunt group name the NAS is a member of. This is done by adding the attribute value pair <Huntgroup-Name,name> to the list of request pairs. 

During request processing the Huntgroup-Name attribute can be checked to make decisions about how to handle the request. For example you may want to restrict the authentication mechansim based on the type of NAS.

## Using unlang to emulate huntgroup behaviour in SQL
Traditionally in FreeRADIUS huntgroups were implemented in the preprocess module (rlm_preprocess), which on start up, read the configuration file /etc/raddb/huntgroups to associate each NAS with a huntgroup. This worked well for configurations using flat files, but if your configuration relied heavily on SQL, it was a bit awkward.

With the introduction of ulang in FreeRADIUS 2.0 it is easy to implement huntgroups using SQL XLAT. This example uses MySQL as the backend database, but it's easy to adjust for another SQL server.

### Building the table
Create a huntgroup table where we store each NAS and the huntgroup it is a member of, we'll call this table radhuntgroup.

    CREATE TABLE radhuntgroup (
        id int(11) unsigned NOT NULL auto_increment,
        groupname varchar(64) NOT NULL default '',
        nasipaddress varchar(15) NOT NULL default '',
        nasportid varchar(15) default NULL,
        PRIMARY KEY  (id),
        KEY nasipaddress (nasipaddress)
    ) ;

### Inserting entries
Populate the radhuntgroup table with your NAS information. For our example we'll insert an entry for a NAS whose ip-address is 192.168.0.10 and put it in the huntgroup "private".

    INSERT INTO `radhuntgroup` (groupname, nasipaddress) VALUES ("example", "192.168.0.10");
 
The contents of the table show now look like this

    SELECT * FROM `radhuntgroup`;
       +----+-----------+--------------+-----------+
       | id | groupname | nasipaddress | nasportid |
       +----+-----------+--------------+-----------+
       |  1 | foo       | 192.168.0.10 | NULL      | 
       +----+-----------+--------------+-----------+

### Configuring FreeRADIUS
* Set up your database configuration in sql.conf (if you haven't done so already)
* Locate the ``authorize { }`` section in your radiusd.conf or sites-enabled/defaut configuration.
* After the preprocess module insert the following
      update request {
          Huntgroup-Name := "%{sql:SELECT `groupname` FROM `radhuntgroup` WHERE nasipaddress='%{NAS-IP-Address}'}"
      }


This performs a lookup in the radhuntgroup table using the ip-address as a key to return the huntgroup name. 
It then adds an attribute/value pair to the request where the name of the attribute is Huntgroup-Name and it's value is whatever was returned from the SQL query. 

If the query did not find anything then the value is the empty string. You can check for this using the unlang statement ``if(Huntgroup-Name == ''){``.

# More examples
## Combining with SQL authorisation
Suppose you want to only allow the group **site_a_admins** to be used when logging into a NAS' at **site_a** .

Assuming **example_user** was already a member of **site_a_admins**, you would follow the steps below.

    SELECT * FROM `radusergroup`;
       +--------------+---------------+----------+
       | username     | groupname     | priority |
       +--------------+---------------+----------+
       | example_user | site_a_admins |        0 | 
       +---------------+---------------+----------+

First add a list of the IP addresses of NAS' at **site_a** into the `radhuntgroup` table.

    INSERT INTO `radhuntgroup` (groupname, nasipaddress) VALUES ("site_a", "192.168.0.10");
    INSERT INTO `radhuntgroup` (groupname, nasipaddress) VALUES ("site_a", "192.168.0.11");
    INSERT INTO `radhuntgroup` (groupname, nasipaddress) VALUES ("site_a", "192.168.0.12");

Now add an entry in the `radgroupcheck` table which says for the **site_a_admins** group only matches, when a Huntgroup-Name attribute with a value of **site_a** exists in the request.
@todo insert query example

    SELECT * FROM `radgroupcheck`
          +----+----------------+----------------+----+----------+
          | id | groupname      | attribute      | op | value    |
          +----+----------------+----------------+----+----------+
          |  1 | site_a_admins  | Huntgroup-Name | == | site_a   | 
          +----+----------------+----------------+----+----------+

@todo reply table example

### Example to allow users in Groups/Profiles access to different NAS.
This below code was described by the friendly people at the Freeradius-mailinglist. Many people have looked for a solution like this before regarding denying access or allowing access to NAS-servers in a Huntgroup matching against Rad-users group/profile membership. This is to be put in the (Ubuntu) /etc/freeradius/sites-available/default config file or equvivalent. This example allows Radius users in the Group 1 and 2 to login to access NAS in the Huntgroup 1 and 2. Please note the security concern that users thats not in any huntgroup will be able to access any device, so make a group "restricted" and place any users there to prohibit access.

        update request {
                Huntgroup-Name := "%{sql:SELECT groupname FROM radhuntgroup WHERE nasipaddress='%{NAS-IP-Address}'}"
        }
        # only allow Groupname1 to Groupname1 and Groupename2
          if (SQL-Group == "Groupname1") {
            if (Huntgroup-Name != "Groupname1" && Huntgroup-Name != "Groupname2") {
              reject
            }
          }
        # allow Groupname3 to only access Groupname1 and Groupname2 and Groupname3
         elsif (SQL-Group == "3rdline") {
            if (Huntgroup-Name != "Groupname1" && Huntgroup-Name != "Groupname2" && Huntgroup-Name != "Groupname3") {
              reject
            }
          }
        # else user can by default access everything except "restricted"
          else {
            if (Huntgroup-Name == "restricted") {
              reject
            }
          }

This above works assuming the NAS and Huntgroups is set up correctly. Any user must be in the right Group/Profile, NAS must be connected to the right Huntgroups etc. Dont forget to restart the freeradius server, or use the debug Freeradius -X to debug.

***

1. A new request arrives, in the request are the following attribute/value pairs (along with other attribute/value pairs)
      User-Name = "example_user"
      NAS-IP-Address = 192.168.0.10
1. The authorize section executes and the "update request" we added performs a SQL query on the `radhuntgroup table`. The variable `%{NAS-IP-Address}` is replaced with the value of NAS-IP-Address in the request.
1. SQL XLAT query runs and matches the first row in our `radhuntgroup` table and returns **site_a** as the huntgroup name. The request is then updated with the attribute/value pair ``Huntgroup-Name = "site_a"``.
1. SQL modules runs. If group checking is enabled the first thing it does is lookup the user name in the radusergroup table. In our example **example_user** is looked up and is found to belong to the group **site_a_admins**.
* SQL group check runs. The `radgroupcheck` table is consulted; for every group the user is a member of a list of <attribute,operator,value> tuples are returned. These tuples are then compared to the attribute/value pairs in the request using the operator specified.
* If all the check items match, the `radgroupreply` table is consulted, and all attributes listed there for the group are added to the reply.