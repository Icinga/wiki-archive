#Icinga IDOUtils Testing
##Icinga IDOUtils Tests
###Test - but what exactly?
Icinga IDOUtils are a submodule of Icinga Core, so installation tests remain basically the same, but with other focus.
###Building the source
####Configure
	./configure --enable-idoutils
####Make
	make idoutils shows warnings/errors?
####Make Install

* make install-idoutils does install all files needed to run?
* configs are not overwritten, check on the sample ones.

###Check Configuration
If asked provide the configuration files of idomod.cfg and ido2db.cfg for example using this command (adjust /etc to your possible prefix)

	# find /etc -type f -name 'ido???.cfg'| xargs egrep -v "^$|^#"
	
Socket type and location must match!

	/etc/icinga/idomod.cfg:output_type=unixsocket
	/etc/icinga/idomod.cfg:output=/var/icinga/ido.sock
	/etc/icinga/ido2db.cfg:socket_type=unix
	/etc/icinga/ido2db.cfg:socket_name=/var/icinga/ido.sock
 
###Debugging

ido2db.cfg and idomod.cfg provide debug settings. Set them to those values to get everything:

* debug_level=-1 (everything)
* debug_verbosity=2 (very detailed)
* debug_file=/where/you/have/enough/storage/ido2db.debug / idomod.debug
* max_debug_file_size=1000000000 (1GB)


####idomod
The output is rather larger as it's directly the functions being called when new data gets there through from the core. Afterwards this data is being processed on the socket.

####ido2db
The debugfile only get's written, if iodmod connects correctly through the socket. So if it's not updated, this is a reminder, that there's something wrong with idomod or the socket being created/used.

###Errors
IDOUtils consisting of idomod and ido2db can be setup in different combinations.
#### idomod and ido2db

* idomod loaded, see syslog/icinga.log

```
# cat /var/log/messages | grep ido | less
# cat /var/log/messages | grep icinga | less
```
 
* verify that idomod is connected via socket to ido2db - check running processes and count the number.

```
# ps aux | grep ido2db | grep -v grep | wc -l
``` 

* must be equal to 2 (1 parent, 1 connected idomod)
* if not - check logs, debug logs for data inserts - see Icinga IDOUtils Testing for log location

*Using Debian/Ubuntu packages, check that ido2db is set to 'yes' in /etc/default/icinga
verify that idomod is enabled in your configuration*
 

*You can enable **idomod** using the module definition **OR** icinga.cfg:broker_module.
Make sure that there's only **one** enabled, and not 2 modules racing for performance!

**After enabling, restart Icinga Core to load idomod**. Check that via syslog, ido2db will tell when idomod is successfully connected! (example see below)*

```
# grep broker_module /etc/icinga/icinga.cfg
# grep modules /etc/icinga/icinga.cfg
# grep idomod /etc/icinga/modules/idoutils.cfg
```

#### Check IDOUtils DB

* Check the credentials defined in databases.xml
* login via shell client, or similar

```
# mysql -u <icingauser> -p <icingadb>
Enter Password: <icingapass>
``` 

* check status, config tables - http://docs.icinga.org/latest/en/db_model.html

**More data integrity tests can be found here:** [Special IDOUtils Queries]()
 
* make sure the data is available
	* see sections Configs and Status below

#### check if idomod is connected

* ido2db will log to syslog when a client is connected

```
# cat /var/log/messages | grep ido2db
 
Apr 19 14:06:10 icinga-dev ido2db: Client connected, data available.
Apr 19 14:06:10 icinga-dev ido2db: Handling client connection...
Apr 19 14:06:10 icinga-dev ido2db: Successfully connected to mysql database
``` 

* ido2db will insert a successful connection attempt from idomod into the conninfo table.
	
`mysql> select * from icinga_conninfo order by connect_time desc limit 2;`
 
* If that data is not updated, make sure idomod is loaded into the Core (check syslog, icinga.log for idomod entries - more above)!

```
# cat /var/log/messages | grep ido | less
# cat /var/log/messages | grep icinga | less
# grep broker_module /etc/icinga/icinga.cfg
# grep modules /etc/icinga/icinga.cfg
# grep idomod /etc/icinga/modules/idoutils.cfg
```

#### check that ido2db processes data

* set debug_level=-1 in ido2db.cfg and restart ido2db and icinga (in this order)

```
# vim /etc/icinga/ido2db.cfg 
debug_level=-1 
# service ido2db restart
# service icinga restart
```

* tail the debug log file (location in ido2db.cfg)

```
# tail -f /var/log/icinga/ido2db.debug
```

#### Sockets
Set different sockets in idomod.cfg and ido2db.cfg, but make sure they match.

Also if they don't check on the erorr messages and look forward to "how to improve the error messages"?

* Unix socket both on local machine, database local too
* Unix socket both on local machine, database extern
* TCP socket (with and without SSL) idomod and ido2db local, db local or extern
* TCP socket (with and without SSL) idomod local and ido2db extern, db extern

#### Schema version

The Schema version should at least match the major (so e.g. schema 1.7.0 will be usable with 1.7.x of Icinga IDOUtils).

	mysql> select * from icinga_dbversion;

#### Connection

The database connection could be limited. E.g. Postgresql needs adapted settings in pg_hba.conf for IPv4 and IPv6 in order to allow the ido2db daemon to connect to the database.

ido2db will insert a succesful connection attempt from idomod into the conninfo table.
	
	mysql> select * from icinga_conninfo order by connect_time desc limit 2;

#### Databases

Configure different databases in the background and apply different settings on ido2db.cfg (create e.g. ido2db.cfg-pgsql and just copying them over as new ido2db.cfg).

* MySQL
* Postgres
* Oracle

[IDOUtils Database Upgrade Testing]()

###Data

**More data integrity tests are collected here**: [Special IDOUtils Queries]()

####Initial connection

* check if connection to database is established
* check on the table icinga_instances, if the same name is in idomod.cfg is there
* check icinga_conninfo if idomod and idomod trimming thread left their entries


####Config data

* Stop ido2db, stop Icinga
* Start ido2db, start Icinga
* check syslog if everythings fine
* check tables icinga_services/hosts/contacts //groups etc if the information matches
* check icinga_objects for matching objects from your config


##### Active Hosts
	mysql> select * from icinga_hosts join icinga_objects on host_object_id=object_id where name1='yourhostname' and is_active=1;

##### Active Services by config_type
```
mysql> select config_type, is_active, count(*) as cnt from icinga_services join icinga_objects on service_object_id=object_id group by config_type, is_active;
+-------------+-----------+------+
| config_type | is_active | cnt  |
+-------------+-----------+------+
|           0 |         1 | 2002 |
+-------------+-----------+------+
1 row in set (0.01 sec)
```
#### Actual state information

* check icinga_host/servicestatus for the hostinformation and compare to those from status.dat and the cgis
* check the debug log for updates happening (insert_or_update functions on those tables

#### Historical Data
* Check if all rows get correctly updated
* Check over period if all checks get into the database

##### Other data
* Check if output and long_output are being written

### Queries

* Check on the RDBMS if there are queries causing performance issues
* do queries fail or produce strange output (what you might get through the database or the debug log)
* compare MySQL, Postgres and Oracle regarding their performance

## Found something?
Then please let us know: [Create an Issue]()
## What else?

Next to Icinga Core there are projects in Icinga, which needs to be tested altogether then.
Have a look into
[Icinga Core Testing]()

[Icinga API Testing]()

[Icinga Web Testing]()

[Icinga Doc Testing]()