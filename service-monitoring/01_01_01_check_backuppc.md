# check_backuppc

## Vorbereitungen
* NRPE installed


## Plugin

http://n-backuppc.cvs.sourceforge.net/viewvc/n-backuppc/check_backuppc/check_backuppc?revision=1.21

```
# apt-get install libnagios-plugin-perl

# vim check_backuppc

-use lib "NAGIOS_LIB";
+use lib "/usr/lib/nagios/plugins/";
-use lib "BACKUPPC_LIB";
+use lib "/usr/share/backuppc/lib/";
```

A corrected Version for Debian: [check_backuppc](attachments/check_backuppc)

##  Config

### sudo

```
# visudo

nagios  ALL= (backuppc) NOPASSWD: /usr/lib/nagios/plugins/check_backuppc
```

### nrpe commands

```
# vim /etc/nagios/nrpe.d/icinga.cfg

# our special commands

command[check_backuppc]=/usr/bin/sudo -u backuppc /usr/lib/nagios/plugins/check_backuppc -w 1 -c 2
command[check_backuppc_host]=/usr/bin/sudo -u backuppc /usr/lib/nagios/plugins/check_backuppc -H $ARG1$ -w 1 -c 2
```

### icinga commands

```
# special commands for backup checks, targetting one backup host, but asking for hosts
define  command {
        command_name    check_nrpe_backup
        command_line    $USER1$/check_nrpe -H backuphost -t 30 -c check_backuppc
}
define  command {
        command_name    check_nrpe_backup_host
        command_line    $USER1$/check_nrpe -H backuphost -t 30 -c check_backuppc_host -a $HOSTNAME$
}
```

### icinga checks
Get a current list which backup hosts are configured.

```
# cat /etc/backuppc/hosts | awk '{print $1}' | egrep -v "^#|^$|host"
```

depending on that list, configure your icinga.

```
  ( ( "check_nrpe_backup", "BACKUPPC all", True), [ "backuppc-all" ], ALL_HOSTS ),
  ( ( "check_nrpe_backup_host", "BACKUPPC", True ) , [ "backup-pc" ], ALL_HOSTS ),
```

### icinga notes_url
If there's an error, directly link to it.

```
extra_service_conf["notes_url"] = [
....
# Backuppc
( "http://host/backuppc/", [ "backup-pc" ], ALL_HOSTS, [ "BACKUPPC" ] ), # "
]
```
