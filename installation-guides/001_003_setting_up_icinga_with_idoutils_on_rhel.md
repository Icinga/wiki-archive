#Setting up Icinga with IDOUtils on RHEL

**These guides were written targetting Icinga 1.x.**

**This applies to Fedora and CentOS as well.**

Package paths may differ! [Check Package Specific Locations]()


## General
What will you get once Icinga with IDOUtils using this guide is installed?

* Icinga Core
* Icinga Classic UI (the "CGIs")
* Icinga IDOUtils
	* Mysql or Postgresql as database
* Icinga Docs

If you came here looking for Icinga Web Setup, please check: Setting up Icinga Web on RHEL.

Packages are preferred for upgrading and maintenance easyness. Currently, there's only a bug open to bring Icinga upstream into EPEL - [Bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=693608) (Warnung) Packagers welcome!

It's the nature of packages that they can take a bit longer because testing and so on needs to be run on various procedures and platform.


If you can't wait that long, build your own rpms here: [Build Icinga RPMs]()

## Requirements
For oracle, you will need ocilib instead of libdbi, and a special configure flag for the packages themselves. This is not described with this guide.

Follow the instructions on how to adde repoforge here: http://repoforge.org/use/ or http://wiki.centos.org/AdditionalResources/Repositories/RPMForge

After adding repoforge, you can invoke a simple search for yum with

`# yum search icinga`

## Setup
```
# yum install icinga icinga-gui icinga-doc icinga-idoutils-libdbi-mysql
or
# yum install icinga icinga-gui icinga-doc icinga-idoutils-libdbi-pgsql
```


Do not forget to install the Nagios Plugins

`# yum install nagios-plugins`

### IDOUtils

After having installed the packages, make sure to setup your database accordingly. The icinga-idoutils package provides the libdbi database abstraction layer, where you need to install the according drivers then.


Icinga 1.7.0 will require libdbi-dbd-* in their defined package.
This will of course require a valid MySQL or Postgresql Server to be already installed.

```
# yum install mysql-server mysql-client libdbi libdbi-devel libdbi-drivers libdbi-dbd-mysql
or
# yum install postgresql postgresql-server libdbi libdbi-devel libdbi-drivers libdbi-dbd-pgsql
```

In Cent6 the mysql-client package has been renamed to mysql
The SQL schema scripts are located in` /usr/share/doc/icinga-idoutils-libdbi-*/db` (1.7.0), please read the `README.RHEL.idoutils` for more information.

`# less /usr/share/doc/icinga-idoutils-libdbi-*/README.RHEL.idoutils`

#### MySQL

```
# mysql

mysql> CREATE DATABASE icinga;

mysql> GRANT USAGE ON icinga.* TO 'icinga'@'localhost' IDENTIFIED BY 'icinga' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0;

mysql> GRANT SELECT, INSERT, UPDATE, DELETE, DROP, CREATE VIEW, INDEX, EXECUTE ON icinga.* TO 'icinga'@'localhost';

mysql> FLUSH PRIVILEGES;

mysql> quit
```

Now import the installed schema, see `/usr/share/doc/icinga-idoutils-libdbi-*-$version/db/mysql`

```
# cd /path/to/idoutils/sqls
# mysql icinga < mysql.sql
```

#### Postgresql

```
# su - postgres

postgres@icinga-dev:~$ psql template1
psql (9.1.2)
Type "help" for help.
template1=# create database icinga;
CREATE DATABASE
template1=# \q

postgres@icinga-dev:~$ createlang plpgsql icinga

postgres@icinga-dev:~$ psql
psql (9.1.2)
Type "help" for help.
postgres=# create role icinga;
CREATE ROLE
postgres=# alter role icinga login;
ALTER ROLE
postgres=# grant all on database icinga to icinga;
GRANT
postgres=# \q
```

Now import the installed schema, see `/usr/share/doc/icinga-idoutils-*-$version/db/pgsql`

```
postgres@icinga-dev:~$ cd /path/to/idoutils/sqls
postgres@icinga-dev:~$ psql -U icinga -d icinga < pgsql.sql
```

Now that Postgresql does use a local user to be trusted (insecure, but good as startup), edit pg_hba.conf accordingly. See Postgresql Manual for a more advanced setup on user auth and privilegues.

```
# vim /var/lib/pgsql/data/pg_hba.conf

# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD

# "local" is for Unix domain socket connections only
local   all         all                               trust
# IPv4 local connections:
host    all         all         127.0.0.1/32          trust
# IPv6 local connections:
host    all         all         ::1/128               trust

#icinga
local    icinga     icinga                    trust
host    icinga          icinga          127.0.0.1/32            trust
host    icinga          icinga          ::1/128                 trust
```

and reload the Postgresql server.

### Config

There's not much to do anymore. The main config tree is located in

	# /etc/icinga/

For further object config proceed to official docs.

### SELinux
**Disable** it or submit a proper selinux policy.

```
# setenforce 0

# sed -i s/enforcing/permissive/ /etc/selinux/config
```

### GUI
Default login icingaadmin:icingaadmin - new users can be added with

	# htpasswd /etc/icinga/passwd youradmin


### IDOUtils

**DO NOT EDIT icinga.cfg for broker_module entry!!! Icinga RPMs will use the /etc/icinga/modules/idoutils.cfg with the module definition automatically. Defining that twice can lead into strange errors!**

Verify that by looking into the modules/idoutils.cfg file

**Icinga 1.7.0 Changes: /usr/bin/idomod.o will be /usr/lib64/icinga/idomod.so**

```
# vim /etc/icinga/modules/idoutils.cfg

define module{
        module_name     idomod
        module_type     neb
        path            /usr/lib64/icinga/idomod.so
        args            config_file=/etc/icinga/idomod.cfg
       }
```

Other event broker modules can be defined using this module definition as well.

#### idomod.cfg
Decide wether to use unix or tcp sockets, as well as some more as described in [Optimize IDOUtils performance]()

#### ido2db.cfg

Edit your database credentials here, if changed above on creation.

```
db_servertype=mysql
#db_servertype=pgsql
db_host=localhost
db_port=3306
#db_port=5432
db_name=icinga
db_prefix=icinga_
db_user=icinga
db_pass=icinga
```

### Upgrading

```
# yum install icinga icinga-gui icinga-doc icinga-idoutils-libdbi-mysql
or
# yum install icinga icinga-gui icinga-doc icinga-idoutils-libdbi-pgsql
```

### IDOUtils

There is no db upgrade script like in Debian, so you need to keep track of that yourself.

First, get the schema version

```
# mysql icinga

mysql> SELECT * FROM icinga_dbversion;
+--------------+----------+---------+---------------------+---------------------+
| dbversion_id | name     | version | create_time         | modify_time         |
+--------------+----------+---------+---------------------+---------------------+
|            1 | idoutils | 1.7.0   | 2012-05-14 17:46:50 | 2012-05-14 17:46:50 |
+--------------+----------+---------+---------------------+---------------------+
1 row in set (0.00 sec)
```

and then decide, what to do - based on the official upgrade docs here: http://docs.icinga.org/latest/en/upgrading_idoutils.html
i.e. upgrading to version 1.7.0 is just one incremential step, so this would mean to apply the now installed upgrade script.

```
# mysql icinga < mysql-upgrade-1.7.0.sql
<verify upgrade ok>
```

Remember, that upgrades needs to be done incremental, so if there will be 1.8.0 from 1.6.0. you need to apply both steps.

```
# mysql icinga < mysql-upgrade-1.7.0.sql
<verify upgrade ok>
# mysql icinga < mysql-upgrade-1.8.0.sql
<verify upgrade ok>
```