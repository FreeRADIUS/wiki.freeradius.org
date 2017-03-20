# Async SQL

Based on comments from Philippe Wooding.

The SQL module needs to be async.  Which involves a lot of changes.  Philippe has start on changes https://github.com/pwdng/freeradius-server/tree/async_sql

## Xlats

The underlying [xlat](/version4/xlat) code is still synchronous.  That needs to be updated before the SQL module will be full async.

## paircompare()

The [paircompare](/version4/paircompare) callbacks (e.g. `SQL-Group`) are still sync

## NAS from SQL

This is done during the instantiation phase of the module.  We would need major changes to make it async.

After some discussion, it's probably best to remove the "read clients" code from SQL.

i.e. the "NAS from SQL" functionality should be replaced by the "dynamic clients" code.  We now have the `map` functionality in the server, so NAS information can be read from the `nas` table.  That should simplify the module.

The async nature of v4 means that the server should be able to accept a large number of packets from unknown clients, and run those through the "dynamic clients" code.

The requirement here is that the v4 network stack be able to track multiple outstanding packets from "unknown" clients.  In v3, it is hard-coded to only allow one packet from an unknown client.