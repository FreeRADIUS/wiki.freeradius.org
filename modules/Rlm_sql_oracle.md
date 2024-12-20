This is the driver of the [[rlm_sql]] [[module]] that connects to Oracle [[SQL]] databases.

## Manually building the rlm_sql_oracle module - Oracle Instant Client 11 64bit
This procedure can be used where an rlm_sql_oracle package is not available, and you're using a package management system to install the server.

1. Download and install the Oracle instant client
2. Download and decompress the FreeRADIUS src tarball version that matches your installed package
3. Run ``configure`` with oracle-specific arguments.  You may need to change the paths to ones which are specific to your system and/or the Oracle version.
    * ``./configure --with-oracle-lib-dir=/usr/lib/oracle/11.2/client64/lib --with-oracle-include-dir=/usr/lib/oracle/11.2/client64``
    * ``make``.  You should see it making ``rlm_sql_oracle-<fr_version>.so``
    * ``cd src/modules/rlm_sql/drivers/rlm_sql_oracle``
    * ``make install``
That will install ONLY the Oracle module.  If you don't see it building or installing anything, edit the ``Makefile`` in the ``rlm_sql_oracle`` directory, and fix it to point to the correct directories.  You do not need to re-run ``configure``
4. ``vi /etc/raddb/modules sql``
    *     :%s/database = ".*"/database = "oracle"/g
5. From the decompressed archive ``cp -r raddb/sql/oracle /etc/raddb/sql/``
6. Ensure that the environmental variable ``LD_LIBRARY_PATH`` contains the path ``/usr/lib/oracle/11.2/client64/lib`` and that ``ORACLE_HOME``  contains the path ``/usr/include/oracle/11.2/client64``, if they don't add them in the /etc/init.d/radiusd file
7. Launch FreeRADIUS ``radiusd -X`` for debug and if everything looks ok start the service

## See also
* [[Oracle DDL script]]