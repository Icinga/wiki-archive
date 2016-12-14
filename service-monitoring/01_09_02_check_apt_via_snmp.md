# check_apt via SNMP

## Clients

```
# apt-get install nagios-plugins
# vim /etc/snmp/snmpd.conf
```

```
exec aptupdate /usr/lib/nagios/plugins/check_apt
```

## Icinga

```
# vim /usr/local/icinga/libexec/check_apt_via_snmp

#!/bin/bash

OUTPUT=`/usr/local/icinga/libexec/check_snmp -H $1 -t30 -C COMMUNITY -o .1.3.6.1.4.1.2021.8.1.101.1 -r "APT OK"`
STATE=${PIPESTATUS[0]}
echo ${OUTPUT#SNMP}
# everything !ok is warning only
if [ $STATE == 0 ]; then
        exit 0
else
        exit 1
fi
```

## Test
`/usr/local/icinga/libexec/check_apt_via_snmp your.hostname`

## Config

```
define command {
        command_name    check_apt_snmp
        command_line    $USER1$/check_apt_via_snmp $HOSTADDRESS$
}
```

```
define service {

...
        check_command  check_apt_snmp
...
}
```
