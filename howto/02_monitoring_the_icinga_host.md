#Identify long lasting checks
##Why?
Actually, the Icinga host itsself should be monitored as well not to finally run into some predictable issues (like logs on /var getting too big). 

Furthermore, there are more (business) critical things to monitor as well such as backends (rrds, database, etc)

Another hint is to use a secondary instance to monitor the Icinga master (need not be another Icinga instance, can be some manual check scripts, cron and mail as well for alerting).

##How?

By using the default Plugins of your choice (those actually provided) plus external for special service checks.

##What?

If you are already using snmp, use it local as well and fetch e.g. filesystems, procs, interface through snmp checks :-)

|Type|Specific|Plugins|
|---|---|---|
|System| Filesystem|check_disk|
||CPU (Load, Usage)|check_load|
||Memory (RAM, Swap)|check_swap|
||Hardware|check_hpasm|
||Total procs|check_procs|
||System Activity Reports| check_sar_perf |
||Disk performance|check_disk_performance|
||I/O|check_iostat, check_sar_perf|
||Interfaces|check_snmp, check_interface|
||Users|check_users|
||Logs|check_logfiles|
||NTP|check_ntp_peer|
||Updates|check_apt, check_yum|
|Icinga|Actual Status|check_nagios|
||Performance Statistics|check_icingastats|
|Database|MySQL, Postgresql, Oracle|check_mysql_health, check_postgres, check_oracle_health|
||advanced queries for data integration|Icinga IDOUtils Testing| 
||housekeeping checks|check_mysql_health, check_postgres, check_oracle_health|
|IDOUtils|ido2db process check|check_procs (Icinga sample config ships "ido2db Process")|
||Icinga Core sends data|Special IDOUtils Queries|
|Reporting|Tomcat Jasperserver 8080|check_http|
|PNP|NPCD, rrdcached process check|check_procs|
||perfdata file check (ramdisk)|check_disk, check_file_age|
||stale RRD checks|check_pnp_rrds.pl|
|GUI|Webserver|check_http|
||Certificates|check_http|
||Authorization|check_http|
|Notifications|Mailing (Queue)|check_mailq|
||SMS (GSM modem)|check_sms3status| 
||XMPP, Twitter, IRC, etc.|todo|

##Specials

If you want to monitor the actual processing of data, use the nagios plugins provided "check_nagios" which works without problems with Icinga as well.

This can be easily called remotly with different access methods (snmp, ssh, nrpe).

A slightly different approach could be check_snmp_icinga too.