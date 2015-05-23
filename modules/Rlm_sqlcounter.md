# rlm_sqlcounter
The **rlm_sqlcounter** module enables a packet counter using accounting records written into an SQL database.

Note: You must use SQL accounting for rlm_sqlcounter to work (list ``sql`` in the ``accounting {}`` section of your virtual server).

## Configuration

* check_name - This is the name of the attribute that is used to set the counter limit.
* sql_module_instance - The instance of the rlm_sql that you wish to use to query.
* key - The key field in the sql lookup - This is usually ``User-Name``.
* The 'reset' parameter defines when the counters are all reset to zero.  It can be hourly, daily, weekly, monthly or never.  It can also be user defined. It should be of the form: it's version 3.0.7 
- num[hdwm] where:
- h: hours, d: days, w: weeks, m: months
 * If the letter is ommited days will be assumed. In example:
- reset = 10h (reset every 10 hours)
- reset = 12  (reset every 12 days)

* query - The sql query that is executed. The resulting value is compared with the attribute ``check_name``.

## Example Setup
The following ``rlm_sqlcounter`` instances are what are referred to by the rest of this document, they may differ from those bundled with the server, but the principle and operations will be similar.

```
sqlcounter noresetcounter {
    counter_name = 'Max-All-Session-Time'
    check_name = 'Max-All-Session'
    sql_module_instance = sql
    key = 'User-Name'
    reset = never
    query = "SELECT SUM(AcctSessionTime) FROM radacct WHERE UserName='%{${key}}'"
}
```

```
sqlcounter dailycounter {
    counter-name = 'Daily-Session-Time'
    check-name = 'Max-Daily-Session'
    sql_module_instance = 'sql'
    key = 'User-Name'
    reset = daily
    query = "SELECT SUM(AcctSessionTime - GREATEST((%b - UNIX_TIMESTAMP(AcctStartTime)), 0)) FROM radacct WHERE UserName='%{${key}}' AND UNIX_TIMESTAMP(AcctStartTime) + AcctSessionTime > '%b'"
}
```

Note: If you are running postgres 7.x, you may not have a GREATER function. A substitute user defined stored ``plpgsql`` procedure would be:
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
```

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
    dailycounter
}
```

```
test0002  Max-Daily-Session := 10800, Cleartext-Password := "blah"
    Service-Type = Framed-User
```

#### SQL
```
authorize {
    sql
    dailycounter
}
```

```sql
INSERT into radcheck VALUES ('','test0002','Max-Daily-Session','10800',':=');
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