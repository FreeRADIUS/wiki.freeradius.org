This is the driver of the [[rlm_sql]] [[module]] that connects to Oracle [[SQL]] databases.

## Manually building the rlm_sql_oracle module - Oracle Instant Client 11 64bit
This procedure can be used where an rlm_sql_oracle package is not available, and you're using a package management system to install the server.

1. Download and install the Oracle instant client
2. Download and decompress the FreeRADIUS src tarball
3. In src/modules/rlm_sql/drivers/rlm_sql_oracle
    * Run ``autoconf``
    * ``./configure --with-oracle-lib-dir=/usr/lib/oracle/11.2/client64/lib --with-oracle-include-dir=/usr/lib/oracle/11.2/client64``
    * ``make`` and you should have a ./libs directory with a file rlm_sql_oracle-<fr_version>.so
4.     cp ./libs/rlm_sql_oracle-<fr_version>.so /usr/lib64/freeradius/
6.     ln -s  /usr/lib64/freeradius/rlm_sql_oracle-<fr_version>.so /usr/lib64/freeradius/rlm_sql_oracle.so
6. vi /etc/raddb/modules sql
    *     :%s/database = ".*"/database = "oracle"/g
7. from the decopressed archive cp -r raddb/sql/oracle /etc/raddb/sql/
8. Be sure that the environmental variable ``LD_LIBRARY_PATH`` contains the path ``/usr/lib/oracle/11.2/client64/lib`` and that ORACLE_HOME``  contains the path ``/usr/include/oracle/11.2/client64``, if they don't add them in the /etc/init.d/radiusd file
9. Launch freeradius (``radiusd -X`` for debug and if ok service radiusd start)

## See also
* [[Oracle DDL script]]