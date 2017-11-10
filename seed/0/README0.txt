#### Config database

Files with names ending with either .ldif or .sh in the `LDAP_SEEDDIR0` directory will be applied when the container is run for the first time when there is no config database.
The *.ldif files will be applied to the database by using slapadd and *.sh files will be sourced.

Files in the `LDAP_SEEDDIR0` directory are ignored when a config database is present.

If no .ldif or .sh files can be found in the `LDAP_SEEDDIR0` directory during seeding the file `LDAP_SEEDDIRa/slapd.ldif` will be copied here and applied.

