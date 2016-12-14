# check_icingastats

## General
Detailed documentation on Icinga performance graphing can be found in the official docs: http://docs.icinga.org/latest/en/perfgraphs.html

https://www.monitoringexchange.org/inventory/Check-Plugins/Software/Nagios/check_nagiostats

## Setup
We rename the check plugin to "check_icingastats" to stay unique.

**Warning:** There's been an update to v1.2 lately - note the changed download url!

```
# cd /path/to/your/plugins
# wget https://www.monitoringexchange.org/attachment/download/Check-Plugins/Software/Nagios/check_nagiostats/check_nagiostats -O check_icingastats
# chmod +x check_icingastats
```

Tell it where the icingastats binary lives. If you prefer to keep your command lines clean, you can edit the default setting in the plugin itself:


```
# vim check_icingastats

EXEC="/path/to/your/binary/icingastats"
```

Otherwise, you can change EXEC per command line option, too:

```
command_line    $USER1$/check_icingastats --EXEC /path/to/your/binary/icingastats ...
```

See the output of "check_icingastats --help" for more information on the available options and parameters.


## Config
Define a new command which can then be used as a new active service check.

```
define command{
        command_name    check_icingastats
        command_line    $USER1$/check_icingastats
}
```

and add it to a service as check_command

```
define service {
...
        check_command  check_icingastats
...
}
```

## [PNP](..//installation-guides/03_00_setting_up_pnp_with_icinga.md) Template

```
# cd /path/to/your/pnp/templates
# cp ../templates.dist/nagiostats.php check_icingastats.php
```
