#Optimize IDOUtils performance
This section is meant to be in progress and should give an overview which parts of IDOUtils can be tweaked, where common bottlenecks reside, and last but not least a step into the docs direction on the configs.

**Starting with 1.9 Icinga IDOUtils uses its own [socket queue proxy](https://www.icinga.org/2013/04/15/icinga-core-reload-problems-addressed-in-1-9/), as well as [transactions](https://www.icinga.org/2013/07/23/thanks-to-immobilienscout24/) for MySQL and Postgresql by default.**

##Configuration
###Core
####Event Broker Options
Configuration starts with core event broker options - keep in mind that this setting affect all event broker modules! [LINK](http://www.mail-archive.com/nagios-users@lists.sourceforge.net/msg24002.html)

###idomod.cfg
#### TCP or Unix Socket
#### Buffer
Increasing the buffer size will help in case of a slow ido2db daemon or database on the other side, dropping out. But keep in mind, that setting this value too high will cause more load on core handling live and cached data in sync.
For performance reasons, it's a good idea to put the buffer into a ramdisk reducing file I/O. If using a virtual environment, put it not on the same volume as the checkresults / logs will run into. Side by side to status.dat will fit probably.

	output_buffer_items=
	buffer_file=

#### Reconnect

It may happen that a slow ido2db daemon or database will disconnect the client after timing out on insert/update/delete. The output buffer will then being filled while waiting for a reconnect. In order to allow faster reconnect, you can decrease the default value of reconnecting. But keep in mind, that too less and large values can create performance issues. Play around a bit yourself.

	reconnect_interval=15
	reconnect_warning_interval=15

#### Data Processing Options
Decide what you need. Stay historical or is it just a status database?

* Normal operations
	* drop \*TIMED_EVENT_DATA (default)
* Icinga Web Requirements
	* drop \*TIMED_EVENT_DATA
	* drop \*CHECK_DATA
	* keep \*LOG_DATA
	* keep \*STATUS_DATA
	* keep \*STATE_HISTORY
	* Config tables
* Only Status Data
	* drop \*CHECK_DATA
	* drop \*TIMED_EVENT_DATA
	* drop \*LOG_DATA
* Historical Data
	* watch \*LOG_DATA
	* watch \*CHECK_DATA
		* archive older, static data into seperated tables
		* use the housekeeping cycles in ido2db.cfg
* Only Perfdata
	* drop LOG_DATA
	* keep \*CHECK_DATA

Use the calculator by Consol: [Calculate](http://labs.consol.de/nagios/ndo-data-processing-options/)

```
#define IDOMOD_PROCESS_PROCESS_DATA                   1
#define IDOMOD_PROCESS_TIMED_EVENT_DATA               2
#define IDOMOD_PROCESS_LOG_DATA                       4
#define IDOMOD_PROCESS_SYSTEM_COMMAND_DATA            8
#define IDOMOD_PROCESS_EVENT_HANDLER_DATA             16
#define IDOMOD_PROCESS_NOTIFICATION_DATA              32
#define IDOMOD_PROCESS_SERVICE_CHECK_DATA             64
#define IDOMOD_PROCESS_HOST_CHECK_DATA                128
#define IDOMOD_PROCESS_COMMENT_DATA                   256
#define IDOMOD_PROCESS_DOWNTIME_DATA                  512
#define IDOMOD_PROCESS_FLAPPING_DATA                  1024
#define IDOMOD_PROCESS_PROGRAM_STATUS_DATA            2048
#define IDOMOD_PROCESS_HOST_STATUS_DATA               4096
#define IDOMOD_PROCESS_SERVICE_STATUS_DATA            8192
#define IDOMOD_PROCESS_ADAPTIVE_PROGRAM_DATA          16384
#define IDOMOD_PROCESS_ADAPTIVE_HOST_DATA             32768
#define IDOMOD_PROCESS_ADAPTIVE_SERVICE_DATA          65536
#define IDOMOD_PROCESS_EXTERNAL_COMMAND_DATA          131072
#define IDOMOD_PROCESS_OBJECT_CONFIG_DATA             262144
#define IDOMOD_PROCESS_MAIN_CONFIG_DATA               524288
#define IDOMOD_PROCESS_AGGREGATED_STATUS_DATA         1048576
#define IDOMOD_PROCESS_RETENTION_DATA                 2097152
#define IDOMOD_PROCESS_ACKNOWLEDGEMENT_DATA           4194304
#define IDOMOD_PROCESS_STATECHANGE_DATA               8388608
#define IDOMOD_PROCESS_CONTACT_STATUS_DATA            16777216
#define IDOMOD_PROCESS_ADAPTIVE_CONTACT_DATA          33554432
#
#define IDOMOD_PROCESS_EVERYTHING                     67108863
#
# You may use the Online Calculator by Gerhard Lausser:
# http://labs.consol.de/nagios/ndo-data-processing-options/
# (please note that there is a checkbox for everything!)
#
# The default setting will remove everything not used by default.
#       TIMED_EVENT_DATA        (-2)
#       SERVICE_CHECK_DATA      (-64)
#       HOST_CHECK_DATA         (-128)
#
# 67108863-(2+64+128) = 67108863-194 = 67108669

data_processing_options=67108669

# If you are planning to use NagVis you may want to use the following setting:
#
#data_processing_options=4061953
#
# You may have to experiment in your environment and find the best value yourself!
```
### Config Output Options
This determines what happens on start/restart/reload of the Core, dumping the configuration (main and object, to be seperated in the data processing options). Keep in mind that a reload is just a SIGHUP restart, not dropping memory and values. Internally, IDOMOD will handle this the same fetching the data from the NEB API.

Using Icinga-Web, the default is currently 2 (retained config data), which requires to have retention data to be enabled (by default) and at least being able to read the data once (so start, and reload then, dumping the configs at this stage). This won't take as long as the original config dump as the config is already compiled/assembled into their respective resulting object data.

Taking the original configs, this will add a hook to the core config parser and add each single directive to IDOMOD too, acquiring cpu cycles and generating much data upon (re)start. Icinga-Web can be set to use that data too.

Within the database itsself, the icinga_hosts and icinga_services table defines the config_type column, where

* config_type=1 <==> config_output_options=2 (retained)
* config_type=0 <==> config_output_options=1 (original)

config_type itsself can be used for Icinga-Web config as XML attribute.

```
# CONFIG OUTPUT OPTION
# This option determines what types of configuration data the IDO
# NEB module will dump from Icinga.  Values can be OR'ed together.
# Hint: 2 is the preferred value, doing a restart after initial startup.
#
# Values:
#         0 = Don't dump any configuration information
#         1 = Dump only original config (from config files)
#         2 = Dump config only after retained information has been restored
#         3 = Dump both original and retained configuration
config_output_options=2
```

### Debug

Keep in mind that writing a debug file may slow down the module processing data. But it will help in order to analyze what data runs from the core through idomod onto the socket against ido2db.

### ido2db.cfg
#### table trimming options
By default timed events are not processed, because the gotten data would remain rather useless. The rest can be set to be cleaned, and holding only the current data, defined by the max age.

It might be a good idea to reduce this especially for logentries because this table can grow very fast.

**Please keep in mind:** If you change the instance_name in idomod.cfg the historical data will stay within the database. The housekeeping process will only catch columns targetted against the current active instance_id (which will be read from IDOMOD and DB on ido2db startup).

```
## TABLE TRIMMING OPTIONS
# Several database tables containing Icinga event data can become quite large
# over time.  Most admins will want to trim these tables and keep only a
# certain amount of data in them.  The options below are used to specify the
# age (in MINUTES) that data should be allowd to remain in various tables
# before it is deleted.  Using a value of zero (0) for any value means that
# that particular table should NOT be automatically trimmed.
#
# Remember: There are no optimized settings, it depends on your database install,
# number/checkinterval of host/service-checks and your desired time of data
# savings - historical vs live-data. Please keep in mind that low delete
# intervals may interfere with insert/update data from Icinga.

# ***DEFAULT***

# Keep timed events for 1 hour
max_timedevents_age=60

# Keep system commands for 1 day
max_systemcommands_age=1440

# Keep service checks for 1 day
max_servicechecks_age=1440

# Keep host checks for 1 day
max_hostchecks_age=1440

# Keep event handlers for 1 week
max_eventhandlers_age=10080

# Keep external commands for 1 week
max_externalcommands_age=10080

# Keep logentries for 31 days
max_logentries_age=44640

# Keep acknowledgements for 31 days
max_acknowledgements_age=44640
```

#### db trim interval
This option can be used to adjust the overall trimming interval (the time between the dates when the table trimming delete queries are actually run against the database). This value has been increased from 60 to 3600 seconds in Icinga 1.4 in order to only run that every hour (and not too stress the database every minute, performance reasons).

Please keep in mind, that the table trimming options go hand in hand with that settings, so setting e.g. max_logentries_age to 30 minutes will also require the trimming interval zu be set less than 30*60 seconds.

	trim_db_interval=3600

#### clear tables on startup
By default, Host/Service Status and Downtimes are not covered by these settings as they remain in the database itsself. The rest will be cleaned either way - if you require various data surviving a core restart/reload, you can disable those settings:

	clean_realtime_tables_on_core_startup=1
	clean_config_tables_on_core_startup=1

##Database
This is mostly DBA's task, importing the DB schema and applying various database specific performance settings.
### SQL Performance in general
[Use the Index](http://use-the-index-luke.com/3-minute-test)
### MySQL
**There have been reports that 5.5.x performs better than native 5.1.x - upgrade if applicable.** 

[mysql queries mit wireshark analysieren](https://blog.netways.de/2013/06/20/mysql-queries-mit-wireshark-analysieren/)

[mysql performance serie teil 1 hardware](https://blog.netways.de/2008/08/20/mysql-performance-serie-teil-1-hardware/)

[MySQL Internals Optimizer tracing](http://forge.mysql.com/wiki/MySQL_Internals_Optimizer_tracing)

[List Entry](http://www.mail-archive.com/nagios-users@lists.sourceforge.net/msg24847.html)

	-We are using a my.cnf based on the "my-innodb-heavy-4G.cnf" example
	that comes with MySQL.
	-The innodb_flush_log_at_trx_commit setting in that file is "2" (instead
	of 1..the difference in performance is huge, and the trade off in data
	integrity is negligible for us).

[monitoring-portal.org Thread](http://www.monitoring-portal.org/wbb/index.php?page=Thread&postID=89332#post89332)

	innodb_support_xa=0 (reduziert die Anzahl der Disk-Flushes)
	innodb_lock_wait_timeout=5 (Anzahl der Sekunden, welche innodb auf die Freigabe von gelocked Rows wartet)
	transaction-isolation = READ-UNCOMMITTED (zwar kein ACID, aber für die NDO absolut tauglich, da nur einer schreibt.)

[monitoring-portal.org Thread](http://www.monitoring-portal.org/wbb/index.php?page=Thread&postID=89332#post89332)

	Angefangen mit der Verwendung der "my-innodb-heavy-4G.cnf", plus
	innodb_flush_log_at_trx_commit = 0 # war 1
	innodb_lock_wait_timeout=5 # war 120
	innodb_support_xa=0 # neu

#### Bernd's tipps
on slide 11-15

[2012-monitoring-workshop-bernd-erk-icinga.pdf](attachments/2012-monitoring-workshop-bernd-erk-icinga.pdf)

#### Catching erroneous queries

[Blog link](http://www.xaprb.com/blog/2009/11/01/catching-erroneous-queries-without-mysql-proxy/)

### Postgresql
Might be worth a shot, next to the various optimazing possibilities in postgresql.conf

**The values below are examples depending on the server postgres is being installed. run the script to gather you own**

```
# ./pgtune -i /var/lib/pgsql/data/postgresql.conf -o postgresql.conf.pgtune -T Mixed
# diff -ur /var/lib/pgsql/data/postgresql.conf postgresql.conf.pgtune

maintenance_work_mem = 704MB # pgtune wizard 2011-07-13
checkpoint_completion_target = 0.7 # pgtune wizard 2011-07-13
effective_cache_size = 8GB # pgtune wizard 2011-07-13
work_mem = 60MB # pgtune wizard 2011-07-13
wal_buffers = 4MB # pgtune wizard 2011-07-13
checkpoint_segments = 8 # pgtune wizard 2011-07-13
shared_buffers = 2816MB # pgtune wizard 2011-07-13
max_connections = 200 # pgtune wizard 2011-07-13
```

#### pgtune
### Oracle

* If the database is just a status database, drop/disable archive logs.
* set open_cursors to an appropriate value, not the default 50 http://www.orafaq.com/node/758
* if you are going into performance issues, consider setting commit_write to nowait

```
open_cursors=1000
commit_write='batch,nowait'
``