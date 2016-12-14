# check_heartbeat

## Know How

* http://www.monitoring-portal.org/wbb/index.php?page=Thread&postID=88869#post88869
* http://www.monitoring-portal.org/wbb/index.php?page=Thread&postID=107708#post107708
* http://www.monitoring-portal.org/wbb/index.php?page=Thread&postID=90634#post90634

Also available in check_mk, but not askable via SNMP (and we do not want to install yet another agent, if we got NRPE and SNMP).

```
# vim /usr/share/check_mk/checks/heartbeat_nodes
# vim /usr/share/check_mk/checks/drbd
```

* http://www.randombugs.com/linux/howto-monitor-linux-heartbeat-snmp.html
* http://www.freebsdcluster.org/~lasse/files/LINUX-HA-MIB.mib
* Perl, nsca/nrpe:
  * http://www.monitoringexchange.org/inventory/Check-Plugins/Operating-Systems/Linux/check_heartbeat_link
* Perl, hp-ux link:
  * http://www.monitoringexchange.org/inventory/Check-Plugins/Software/Clusters/Snmp-Heartbeat-check
* Bash, SNMP:
  * http://www.monitoringexchange.org/inventory/Check-Plugins/Software/Clusters/Snmp-Heartbeat-check
* Bash, SNMP:
  * http://www.monitoringexchange.org/inventory/Check-Plugins/Software/Clusters/Check_snmp_heartbeat_resources
* More and Alternatives:
  * http://exchange.nagios.org/index.php?option=com_mtree&task=search&Itemid=74&searchword=heartbeat

## Installation

Like [DRBD](01_05_01_check_drbd.md)

```
# cd /usr/local/
# mkdir -p icinga/libexec ; cd icinga/libexec
# wget http://wiki.debuntu.org/w/images/b/b4/Check_heartbeat.sh -O check_heartbeat.sh
# wget https://www.monitoringexchange.org/attachment/download/Check-Plugins/Operating-Systems/Linux/check_heartbeat_link/check_heartbeat_link
# chmod +x check_heartbeat.sh check_heartbeat_link
```

### SNMP

```
# cd /opt/nagios/libexec/
# wget https://www.monitoringexchange.org/attachment/download/Check-Plugins/Software/Clusters/Snmp-Heartbeat-check/check_snmp_heartbeat.sh
# chmod +x check_snmp_heartbeat.sh
```

```
# vim /etc/default/snmpd

# snmpd options (use syslog, close stdin/out/err).
#SNMPDOPTS='-Lsd -Lf /dev/null -u snmp -I -smux -p /var/run/snmpd.pid 127.0.0.1'
SNMPDOPTS='-Lsd -Lf /dev/null -u snmp -I -smux -p /var/run/snmpd.pid'
```

In order to let hbagent as sub agent connect to masterx, the snmpd must be configure as master.

```
master agentx
trap2sink localhost
```

restart snmpd


```
# /etc/init.d/snmpd restart
```

configure /etc/heartbeat/ha.cfg hbagent respawn.

```
# sub agent for snmp respwan hbagent
respawn root /usr/lib/heartbeat/hbagent -d
```

vgl http://books.google.at/books?id=1PEqrGYcGBgC&pg=PT232&lpg=PT232&dq=hbagent+respawn&source=bl&ots=OWzOZTZY6X&sig=h_Rm9Tr3nB1C7BfZVSxVUgGzE6Y&hl=de&ei=xgXATb-GOoTvsgbku9zDBQ&sa=X&oi=book_result&ct=result&resnum=2&ved=0CCgQ6AEwAQ#v=onepage&q=hbagent%20respawn&f=false


## Tests

### SNMP

```
# snmpwalk hostname -v 2c -c xxxx .1.3.6.1.4.1.4682.1
SNMPv2-SMI::enterprises.4682.1.1.0 = Counter32: 3
SNMPv2-SMI::enterprises.4682.1.2.0 = Counter32: 2
SNMPv2-SMI::enterprises.4682.1.3.0 = INTEGER: 3
SNMPv2-SMI::enterprises.4682.1.4.0 = Counter32: 1
```

### check_snmp_heartbeat.sh

```
# ./check_snmp_heartbeat.sh -H host -C xxxxx
HEARTBEAT lost some nodes !
```

### check_snmp_heartbeat_resources

:warning does not work properly

```
# ./check_snmp_heartbeat_resources -H host -C xxxx -R 2
```

### check_heartbeat_link

```
# ./check_heartbeat_link
Heartbeat Link OK: host:eth1:up 1.2.3.4:1.2.3.4:up 
```
