Authorization refers to the granting of specific types of service
(including "no service") to a user, based on their authentication,
what services they are requesting, and the current system state.
Authorization may be based on restrictions, for example
time-of-day restrictions, or physical location restrictions, or
restrictions against multiple logins by the same user.
Authorization determines the nature of the service which is
granted to a user.  Examples of types of service include, but are
not limited to:

  * IP address filtering;
  * address assignment;
  * route assignment;
  * QoS/differential services;
  * bandwidth control/traffic management;
  * compulsory tunneling to a specific endpoint;
  * encryption.

## Authorization methods

FreeRADIUS supports many authorization methods, including:

  * Unlang configuration
  * Local files (cf. [[users]])
  * [[rlm_dbm|Local DB/DBM database]]
  * [[LDAP]] Database
      * Any LDAPv3 compliant directory
      * Novell eDirectory
      * OpenLDAP
      * Sun One Directory Server
  * A locally executed program (like a CGI program)
  * [[rlm_perl|Perl program]]
  * [[rlm_python|Python program]]
  * [[rlm_sql|SQL Database]]
      * [[rlm_sql_oracle|Oracle]]
      * [[rlm_sql_mysql|MySQL]]
      * [[rlm_sql_postgresql|PostgreSQL]]
      * [[rlm_sql_sybase|Sybase]]
      * [[rlm_sql_db2|IBM DB2]]
      * Any [[rlm_sql_iodbc|iODBC]] or [[rlm_sql_unixodbc|unixODBC]] supported database

## See also

  * [[Supported Attributes]]
  * RFC 2865
  * [[AAA]]
  * [[AAAA]]
