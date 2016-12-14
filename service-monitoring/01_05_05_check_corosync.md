# check_corosync

## check_corosync_rings

### NRPE Script.

```
# wget "http://exchange.nagios.org/components/com_mtree/attachment.php?link_id=2397&cf_id=29" -O check_corosync_rings ; chmod +x check_corosync_rings
```

### sudo

```
# visudo

nagios  ALL=NOPASSWD: /usr/sbin/corosync-cfgtool -s
 ```

### NRPE
```
command[check_corosync_rings]=/usr/lib/nagios/plugins/check_corosync_rings
 ```

### Icinga

```
##corosync
  ( ( "check_nrpe_command!check_corosync_rings", "corosync rings", True ) , [ "corosync" ], ALL_HOSTS ),
```
