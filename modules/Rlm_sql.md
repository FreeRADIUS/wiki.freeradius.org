# FreeRADIUS SQL Module (rlm_sql)

The SQL module is composed of two parts: a generic SQL front-end ([[rlm_sql]]), and a series of
database-dependent back-end drivers.

## Back-end Drivers

The various different database systems are supported through the following rlm_sql "drivers":
* [[rlm_sql_db2]] - IBM DB2
* [[rlm_sql_firebird]] - Firebird
* [[rlm_sql_freetds]] - FreeTDS (mssql)
* [[rlm_sql_iodbc]] - iODBC
* [[rlm_sql_mysql]] - MySQL
* [[rlm_sql_oracle]] - Oracle
* [[rlm_sql_postgresql]] - PostgreSQL
* [[rlm_sql_sybase]] - Sybase (mssql)
* [[rlm_sql_unixodbc]] - unixODBC (mssql)

In order to use the database drivers, you **must** install the appropriate client libraries for the
desired database on the FreeRADIUS machine.

The rlm_sql_* drivers are **not** a complete client implementation. Instead, it is a small 'shim'
between the FreeRADIUS rlm_sql module, and the respective client libraries.

## SQL Schema and usage

In general, the [[SQL schema]] mirrors the layout of the '[[users]]' file. So for configuring check
items and reply items, see 'man 5 users', and the examples in the 'users' file.

The SQL module employs two sets of check and reply item tables for processing in the authorization
stage.  One set of tables (radcheck and radreply) are specific to a single user.  The other set of
tables (radgroupcheck and radgroupreply) is used to apply check and reply items to users that are
members of a certain SQL group.  The usergroup table provides the list of groups each user is a
member of along with a priority field to control the order in which groups are processed.

When a request comes into the server and is processed by the SQL module, the flow goes something
like this:

* Search the radcheck table for any check attributes specific to the user
* If check attributes are found, and there's a match, pull the reply items from the radreply table
* for this user and add them to the reply
* Group processing then begins if any of the following conditions are met:
  * The user IS NOT found in radcheck
  * The user IS found in radcheck, but the check items don't match
  * The user IS found in radcheck, the check items DO match AND Fall-Through is set in the radreply table
  * The user IS found in radcheck, the check items DO match AND the read_groups directive is set to 'yes'
* If groups are to be processed for this user, the first thing that is done is the list of groups
  this user is a member of is pulled from the usergroup table ordered by the priority field.  The
  priority field of the usergroup table allows us to control the order in which groups are processed,
  so that we can emulate the ordering in the users file. This can be important in many cases.
  For each group this user is a member of, the corresponding check items are pulled from
  radgroupcheck table and compared with the request.  If there is a match, the reply items for this
  group are pulled from the radgroupreply table and applied.
* Processing continues to the next group IF:
  * There was not a match for the last group's check items OR
  * Fall-Through was set in the last group's reply items (The above is exactly the same as in the
    users file)
* Finally, if the user has a User-Profile attribute set or the Default Profile option is set in the
  sql.conf, then steps 4-6 are repeated for the groups that the profile is a member of.
  
For any fairly complex setup, it is likely that most of the actual processing will be done in the
groups.  In these cases, the user entry in radcheck will be of limited use except for things like
setting the user's password.  So, one might have the following setup:

### radcheck table:
```text
joeuser        Cleartext-Password      :=       somepassword
```

### radreply table:
```text
joeuser        Fall-Through       =        Yes
```

### radgroupcheck table:
Check items for various connection scenarios

### radgroupreply table:
reply items for the groups

### usergroup table:
```text
joeuser      WLANgroup    1(this is the priority)
joeuser      PPPgroup     2
```

In the SQL configuration file are _alt queries, these are called when the first SQL query fails or
doesn't alter (insert, delete, update) any rows in the Database.

## What NOT to do

One of the fields of the SQL schema is named 'op'  This is for the 'operator' used by the
attributes.  e.g.:

```text
Framed-IP-Address  =      1.2.3.4
^ ATTRIBUTE ----^  ^ OP   ^ VALUE
```
If you want the server to be completely mis-configured, and to never do what you want, leave the
'op' field blank!

The reason is that with the op field empty, the server does  not know what you want it to do with
the attribute. Should it be added to the reply? Maybe you wanted to compare the operator to one in
the request? The server simply doesn't know.

So put a value in the field. The value is the string form of the [[Operators|operator]]

### Authentication vs Authorization

Many people ask if they can "authenticate" users to their SQL database however the answer is "You
are asking the wrong question."

An SQL database stores information. An SQL database is NOT an authentication server. The
**only** users who should be able to authenticate themselves to the database
are the people who administer it.  Most administrators do NOT want every user to be able to access
the database, which means that most users will not be able to "authenticate" themselves to the
database.

Instead, the users will have their authorization information (name, password, configuration) stored
in the database.  The configuration files for FreeRADIUS contain a username and password used to
authenticate FreeRADIUS to the SQL server.  (See raddb/sql.conf). Once the FreeRADIUS authentication
server is connected to the SQL database server, then FreeRADIUS can pull user names and passwords
out of the database, and use that information to perform the authentication.

## Operators

See [[Operators]]

## Instances

Just like any other module, multiple instances of the [[rlm_sql]] module can be defined and used
wherever you like.

The default .conf files for the different database types, contain 1 instance without a name like so:

```text
sql {
	...
}
```

You can create multiple named instances like so:

```text
sql sql_instance1 {
	...
}
sql sql_instance2 {
	...
}
```

And then you can use a specific instance in [[radiusd.conf]], like so:

```text
authorize {
	...
	sql_instance1
	...
}

accounting {
	...
	sql_instance1
	sql_instance2
	...
}
```

## SQL xlat
The SQL module now supports SQL queries in [rlm_expr](modules/rlm_expr). That is you can extract  the
value of a single field and use it, either as a check item, a request item or a reply item.
The strings will be of the following form:

```sql
%{sql:SELECT mytable.field1 FROM `mytable` WHERE 1}
```

and you may nest xlat statements within SQL strings:

```sql
%{sql:SELECT mytable.field1 FROM `mytable` WHERE mytable.user = %{User-Name}}
```

In case the returned field is multi valued which value is returned is considered UNDEFINED. If there
are multiple instances of the module, the instance name can be used instead of the string 'sql', to
decide which instance will return the information. The xlat string will be of the form:

```sql
%{instance_name:SELECT mytable.field1 FROM `mytable` WHERE 1}
```
For example:

```sql
%{sql_clients:SELECT mytable.field1 FROM `mytable` WHERE 1}
```

## Virtual Modules

In version 3, any redundant modules with an instance name, e.g.

```text
redundant sql_inst {
	sql1
	sql2
}
```

Can also be referenced in an xlat string expansion.

In version <= 2 they cannot, and the expansion will fail.

## See Also

* [Modules](modules)
