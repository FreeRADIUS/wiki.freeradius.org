FreeRADIUS doesn't have a built-in lockout (disable account after X auth fails in Y seconds) but it's easy to implement.

The following does it using SQL - it assumes Postgresql-style syntax for "datetime" datatypes. Configure SQL, then create a table as follows `create table failed (username text, authdate timestamptz);`

sites-enabled/NAME:

    authorize {
      lockout_check
      ...
    }
    post-auth {
      Post-Auth-Type REJECT {
        lockout_incr
      }
    }

policy.conf:

    lockout_check {
      # checks for 5 failed auths in the last 10 minutes
      update control {
        Tmp-Integer-0 := "%{sql:select count(*) from failed where username='%{User-Name}' and now()-authdate < '10 minutes'}"
      }
      if (control:Tmp-Integer-0 > 5) {
        reject
      }
    }
    lockout_incr {
      update control {
        Tmp-Integer-0 := "%{sql:insert into failed (username,authdate) values ('%{User-Name}', now())}"
        # clean up entries we don't need from the table
        Tmp-Integer-1 := "%{sql:delete from failed where now()-authdate > '1 hour'}"
      }
    }

In some cases, you might want to change the schema - the username might not be visible for anonymous outer EAP identities, for example. In this case, extend the SQL schema and SQL queries to compare on attributes that are available, for example, Calling-Station-Id