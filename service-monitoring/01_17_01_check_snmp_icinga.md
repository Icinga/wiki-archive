# check_snmp_icinga

## Idea
Put a base OID into your snmp tree and report the last check time of the icinga instance.

## Icinga Node

### SNMPD Config

```
# vim /etc/snmp/snmpd.conf

pass .1.3.6.1.4.1.24671.1.1.6 /usr/local/sbin/check_icinga.pl
```

### Script

```
# vim /usr/local/sbin/check_icinga.pl

#!/usr/bin/perl
use strict;
use warnings;
my $baseoid = ".1.3.6.1.4.1.24671.1.1.6";
my $requestoid = $ARGV[1];
my $snmp;
my $lastcheck = 0;
my $status_dat = "/var/spool/icinga/status.dat";
open STATUS, $status_dat;
while (<STATUS>) {
    if ( /last_check=(\d+)/) {
        if ($1 > $lastcheck) {
            $lastcheck = $1;
        }
    }
}
close STATUS;
print $requestoid."\n";
print "INTEGER\n";
print "$lastcheck\n";
```

## Remote SNMP Query

```
# snmpget -c community -v2c hostname .1.3.6.1.4.1.24671.1.1.6
SNMPv2-SMI::enterprises.24671.1.1.6 = INTEGER: 1347020676
```
