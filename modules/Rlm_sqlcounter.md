# rlm_sqlcounter

The **rlm_sqlcounter** module enables a packet counter using [[SQL]].

Make sure to have radiusd running properly under sql and there must be a "sql" entry under accounting{ } section of radiusd.conf

## Configuration

* check-name - This is the name of the radius attribute that describes the limit of the sqlcounter.
* sqlmod-inst - The instance of the rlm_sql that you wish to use to query.
* key - The key field in the sql lookup - this is usually the username.
* reset - Specifies the frequency that the sqlcounter should be reset, possible values are:
  - daily
  - weekly
  - monthly
  - never
* query - The sql expresion that is executed - often from the radacct table. The resulting value can then be compared with the value from check-name.

## Example Setup

* Create a text file called sqlcounter.conf in the same directory where radiusd.conf resides (usually ``/usr/local/etc/raddb``) with the following content (for sqlite):

```
sqlcounter noresetcounter {
    counter-name = Max-All-Session-Time
    check-name = Max-All-Session
    sqlmod-inst = sql
    key = User-Name
    reset = never
    query = "SELECT SUM(AcctSessionTime) FROM radacct WHERE UserName='%{${key}}'"
}
```

```
sqlcounter dailycounter {
    driver = "rlm_sqlcounter"
    counter-name = Daily-Session-Time
    check-name = Max-Daily-Session
    sqlmod-inst = sql
    key = User-Name
    reset = daily
    query = "SELECT SUM(AcctSessionTime - GREATEST((%b - UNIX_TIMESTAMP(AcctStartTime)), 0)) FROM radacct WHERE UserName='%{${key}}' AND UNIX_TIMESTAMP(AcctStartTime) + AcctSessionTime > '%b'"
}
```

```
sqlcounter monthlycounter {
    counter-name = Monthly-Session-Time
    check-name = Max-Monthly-Session
    sqlmod-inst = sql
    key = User-Name
    reset = monthly
    query = "SELECT SUM(AcctSessionTime - GREATEST((%b - UNIX_TIMESTAMP(AcctStartTime)), 0)) FROM radacct WHERE UserName='%{${key}}' AND UNIX_TIMESTAMP(AcctStartTime) + AcctSessionTime > '%b'"
}
```

* If you are using postgresql then the query lines would have to be replaced by the following:

``query = "SELECT SUM(AcctSessionTime) FROM radacct WHERE UserName='%{${key}}'"``

``query = "SELECT SUM(AcctSessionTime - GREATER((%b - AcctStartTime::ABSTIME::INT4), 0)) FROM radacct WHERE UserName='%{${key}}' AND AcctStartTime::ABSTIME::INT4 + AcctSessionTime > '%b'"``

``query = "SELECT SUM(AcctSessionTime - GREATER((%b - AcctStartTime::ABSTIME::INT4), 0)) FROM radacct WHERE UserName='%{${key}}' AND AcctStartTime::ABSTIME::INT4 + AcctSessionTime > '%b'"``

If you are running postgres 7.x, you may not have a GREATER function.  An example of one is:

```sql
CREATE OR REPLACE FUNCTION "greater"(integer, integer) RETURNS integer AS '
    DECLARE
        res INTEGER;
        one INTEGER := 0;
        two INTEGER := 0;
    BEGIN
        one = $1;
        two = $2;
        IF one IS NULL THEN
            one = 0;
        END IF;
        IF two IS NULL THEN
            two = 0;
        END IF;
        IF one > two THEN
            res := one;
        ELSE
            res := two;
        END IF;
        RETURN res;
    END;
' LANGUAGE 'plpgsql';

Include the above file to radiusd.conf by adding a line in modules{ } section
```
    modules {
        $INCLUDE  ${confdir}/sqlcounter.conf
        ...some other entries here...
    }

Make sure to have the sqlcounter names under authorize section like the following:
```
    authorize {
        ...some entries here...
        noresetcounter
        dailycounter
        monthlycounter
    }
```

* noresetcounter - The counter that never resets, can be used for real session-time cumulation 
* dailycounter - The counter that resets everyday, can be used for limiting daily access time (eg. 3 hours a day)
* monthlycounter - The counter that resets monthly, can be used for limiting monthly access time (eg. 50 hours per month)

You can make your own names and directives for resetting the counter by reading the sample sqlcounter configuration in ``raddb/experimental.conf``

## Implementation

Add sqlcounter names to be used into radcheck or radgroupcheck table appropriately for sql. For users file just follow the
example below.

Note: The users in the example below must be able to login normally as the example will only show how to apply sqlcounter 
counters.

## Scenarios

### Lifetime limit
A user with User-Name ``test0001`` has total online time limit of 15 hours (the user can login as many times as needed but can be online for total time of 15 hours (54000 seconds)).

#### Users file
```
authorize {
    files
    noresetcounter
}
```

```
test0001  Max-All-Session := 54000, Cleartext-Password := "blah"
    Service-Type = Framed-User
```

The important part of the above entry is ``Max-All-Session := 54000``, which adds the ``control`` attribute ``Max-All-Session``
which is used by the ``rlm_sqlcounter`` instance ``noresetcounter``, to determine how long a user should be allowed online for.

#### SQL
```
authorize {
    sql
    noresetcounter
}
```

```sql
INSERT into radcheck VALUES ('','test0001','Max-All-Session','54000',':=');
```

#### Unlang
```
authorize {
    if (User-Name == 'test0001') {
        update {
            control:Max-All-Session := 54000
        }
    }
    noresetcounter
}
```

### Daily limit
A user with User-Name ``test0002`` has  online time limit of 3 hours per day.

#### Users file
```
authorize {
    files
    noresetcounter
}
```

```
test0002  Max-All-Session := 10800, Cleartext-Password := "blah"
    Service-Type = Framed-User
```

#### SQL
```
authorize {
    sql
    noresetcounter
}
```

```sql
INSERT into radcheck VALUES ('','test0002','Max-All-Session','10800',':=');
```

#### Unlang
```
authorize {
    if (User-Name == 'test0002') {
        update {
            control:Max-Daily-Session := 10800
        }
    }
    dailycounter
}
'''VERY IMPORTANT'''
Accounting must be done via sql or this will not work.