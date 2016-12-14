# check_drbd

## Know How

### Guides

* http://blog.simon-meggle.de/tutorials/nagiosomd-cluster-mit-pacemakerdrbd-teil1/
* http://blog.simon-meggle.de/tutorials/nagiosomd-cluster-mit-pacemakerdrbd-teil2/
* http://blog.simon-meggle.de/tutorials/nagiosomd-cluster-mit-pacemakerdrbd-teil3/
* http://blog.simon-meggle.de/tutorials/nagiosomd-cluster-mit-pacemakerdrbd-teil4/
* http://blog.simon-meggle.de/tutorials/nagiosomd-cluster-mit-pacemakerdrbd-teil5/
* http://blog.simon-meggle.de/tutorials/nagiosomd-cluster-mit-pacemakerdrbd-teil6/

### Scripts
* Perl, nsca/nrpe:
  * http://www.monitoringexchange.org/inventory/Check-Plugins/Operating-Systems/Linux/check_drbd
* Perl, ssh:
  * http://www.monitoringexchange.org/inventory/Check-Plugins/Operating-Systems/Linux/check_drbd2
* Perl, rather old:
  * http://www.monitoringexchange.org/inventory/Check-Plugins/Software/Clusters/check_drbd-pl

## Installation

### check_drbd

```
# cd /usr/local/
# mkdir -p icinga/libexec ; cd icinga/libexec
# wget https://www.monitoringexchange.org/attachment/download/Check-Plugins/Operating-Systems/Linux/check_drbd/check_drbd
# chmod +x *
```

### check_drbd

```
# cd /usr/local/
# mkdir -p icinga/libexec ; cd icinga/libexec
# wget http://doguet.com/pub/centreon/nagios_libs.pm
# wget https://www.monitoringexchange.org/attachment/download/Check-Plugins/Software/Clusters/check_drbd-pl/check_drbd-pl
# vim check_drbd-pl
#use lib "/app/scripts/nagios";
use lib "/usr/local/icinga/libexec";
# chmod +x *
```

## Tests

### check_drbd

```
pgbaer1:/usr/local/icinga/libexec# ./check_drbd
DRBD OK: Device 0 Primary Connected UpToDate
```

### check_drbd-pl
```
pgbaer1:/usr/local/icinga/libexec# ./check_drbd-pl --role Primary/Secondary
CRITICAL Can't open /tmp/drbdout
```

/tmp/drbdout gibts ned ?!? => **kaputt**
