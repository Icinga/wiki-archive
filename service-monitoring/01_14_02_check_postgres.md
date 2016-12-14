# check_postgres

**Warning:** needs review and howto

## General
http://bucardo.org/wiki/Check_postgres

## About
> **check_postgres** is a script for monitoring various attributes of your database. It is designed to work with Nagios, MRTG, or in standalone scripts.


## Setup
http://bucardo.org/check_postgres/check_postgres.pl.html

```
$ git clone git://bucardo.org/check_postgres.git
```

## Configuration

### Icinga

```
# POSTGRESQL
define  command {
        command_name    check_postgres
        command_line    $USER1$/check_postgres.pl -H $HOSTADDRESS$ -db $ARG1$ -dbuser icinga -dbpass blablub -w $ARG2$ -c $ARG3$ --action $ARG4$
}

define  command {
        command_name    check_postgres_dbstats
        command_line    $USER1$/check_postgres.pl -H $HOSTADDRESS$ -db $ARG1$ -dbuser icinga -dbpass blablub --action dbstats
}
```

### Postgresql

```
CREATE ROLE icinga WITH LOGIN ENCRYPTED PASSWORD 'ur-aerger-geheim';
ALTER role icinga CONNECTION LIMIT 12;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO icinga;
```

The connection limit of 12 is an arbitrary number (for now) and should probably be reduced to 3 or 5


## Perfdata
I've patched the Cacti Output to match the perfdata format. This not clean nor upstream, but it works for performance data providing checks such as the dbstats one.

https://github.com/dnsmichi/check_postgres/tree/icinga-pnp

https://github.com/dnsmichi/check_postgres/commit/6deee57ea8230929e6142d34a12998968c93f30a
