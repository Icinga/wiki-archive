# check_interface

## SNMP
* [Nagios Plugins](../plugins/01_02_nagios_plugins.md)
* [check_snmp](01_03_02_check_snmp.md)

## Local
Sourcing from over here

http://www.monitoring-portal.org/wbb/index.php?page=Thread&threadID=19820


```
#!/bin/sh

IFCONFIG="/sbin/ifconfig"

if [ $# -ne 1 -o "$1" = "" -o "$1" = "-h" ]; then
   echo "Usage: `basename $0` InterfaceName"
   echo "Outputs the traffic and error data from an 'ifconfig InterfaceName'"
   exit 3
fi

DATA="`$IFCONFIG $1 2>/dev/null | grep '^ *[TR]X packet' | \
   awk '{ for (i=1;i<NF;i++) printf " %s_%s",$1,$i; }' | \
   sed -e 's/;/ /g' -e 's/\([A-Za-z0-9_]*\):\([0-9][0-9]*\)/ \1=\2; /g' -e 's/ [^; ][^; ]* //g' -e 's/; */ /g'`"

if [ "$DATA" != "" ]; then
   echo "OK: Data read |$DATA"
   exit 0
else
   echo "CRITICAL: Cannot read counters of interface $1"
   exit 2
fi
```
