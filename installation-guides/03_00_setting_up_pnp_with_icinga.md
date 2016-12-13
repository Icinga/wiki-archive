# Setting up PNP with Icinga

These guides were written targetting Icinga 1.x.


**Important**: If you installed Icinga packages, you should look for pnp4nagios packages too. The paths from a manual install will be different than an FHS compliant package install.
E.g. [Setting up PNP with Icinga on Debian](03_06_setting_up_pnp_with_icinga_on_debian.md)

## General
The official docs can be found at http://docs.pnp4nagios.org/pnp-0.6/start

> PNP is an addon to Nagios which analyzes performance data provided by plugins and stores them automatically into RRD-databases (Round Robin Databases, see RRD Tool).
During development of PNP we set value on easy installation and little maintenance while running it. An administrator should do other things than configure graphing tools.
To achieve this task we focused on using standards. PNP only processes performance data built according to theDeveloper Guidelines for nagios plugins. With this limitation we want to honour the work ofNagios Plugin Developers who stick to the guidelines.
For all of those who are still curious the following documentation is made which should help to ease the access to PNP.

## Pre-Requisites

### Icinga Core
This guide assumes that Icinga has been installed with the defaults into /usr/local/icinga with icinga.cfg in the etc directory.

### RRDTool and Perl Bindings

```
# yum install rrdtool rrdtool-perl
```
```
# apt-get install librrds-perl rrdtool
```

[Enable RRDCached in PNP](03_01_enable_rrdcached_in_pnp.md)

## Install
We will be installing __**PNP with NPCD in Bulk Mode**__ - http://docs.pnp4nagios.org/pnp-0.6/config#bulk_mode_with_npcd

**If you want to match the Debian layout, use**
```
# ./configure --with-nagios-user=icinga --with-nagios-group=icinga --with-layout=debian
```

```
# cd /usr/src
# wget http://downloads.sourceforge.net/project/pnp4nagios/PNP-0.6/pnp4nagios-0.6.19.tar.gz
# tar xzf pnp4nagios-0.6.19.tar.gz ; cd pnp4nagios-0.6.19

# ./configure --with-nagios-user=icinga --with-nagios-group=icinga
# make all
# make install install-webconf install-config install-init
```

## Configs

```
# cd /usr/local/pnp4nagios/etc/
# mv rra.cfg-sample rra.cfg
```
Diff only.

### npcd.cfg

```
user = icinga
group = icinga
log_type = file
log_level = -1
load_threshold = 10.0
```

### config.php
**PNP installs config.$version.php as well as config_local.php for upgrading purposes - be sure to remove those if not needed, as this may override existing settings!**

### PNP Urls
By default, the config.php option nagios_base points to the wrong location, linking the wrong cgis.

Change that accordingly to your relative cgi path, like  

```
$conf['nagios_base'] = "/icinga/cgi-bin";
```

### icinga.cfg
Either edit or append. A sample is provided within
**/usr/local/pnp4nagios/etc/nagios.cfg-sample** (choose "**Bulk / NPCD Mode**")

```
#########################################
# perfdata - PNP
#########################################

process_performance_data=1

host_perfdata_file=/usr/local/pnp4nagios/var/host-perfdata
service_perfdata_file=/usr/local/pnp4nagios/var/service-perfdata

service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$

host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$

service_perfdata_file_mode=a
host_perfdata_file_mode=a

service_perfdata_file_processing_interval=30
host_perfdata_file_processing_interval=30

service_perfdata_file_processing_command=process-service-perfdata-file
host_perfdata_file_processing_command=process-host-perfdata-file
```

### commands.cfg
Either edit or append. A sample is provided within **/usr/local/pnp4nagios/etc/misccommands.cfg-sample** (choose "**Bulk with NPCD Mode**")

```
# pnp
define command{
        command_name    process-service-perfdata-file
        command_line    /bin/mv /usr/local/pnp4nagios/var/service-perfdata /usr/local/pnp4nagios/var/spool/service-perfdata.$TIMET$
}

define command{
        command_name    process-host-perfdata-file
        command_line    /bin/mv /usr/local/pnp4nagios/var/host-perfdata /usr/local/pnp4nagios/var/spool/host-perfdata.$TIMET$
}
```

### Apache
If on Debian/Ubuntu, you'll need to enable mod_rewrite

```
# a2enmod rewrite
# /etc/init.d/apache restart
```

and you should update /etc/apache2/conf.d/pnp4nagios.conf to icinga path for authorization.

## Alternative PNP Mode - Bulk Mode with NPCD and npcdmod
You may figure that there's another mode around, "Bulk Mode with NPCD and npcdmod". This works basically the same as described above, but leaves out the additional perfdata template and command definitions, moving the perfdata files into timestamped files.

This part is being handled by the event broker module npcdmod.o which is loaded into the core memory, handling all performance data updates directly.

Just make sure that icinga.cfg contains

```
process_performance_data=1
```

and includes the modules directory like

```
cfg_dir=/etc/icinga/modules
```

Then verify that npcdmod.o and npcd.cfg path is correct and put a new module object configuration file for the event broker module into e.g. /etc/icinga/modules/pnp.cfg

```
# vim /etc/icinga/modules/pnp.cfg

define module{
        module_name     npcdmod
        module_type     neb
        path            /usr/lib/pnp4nagios/npcdmod.o
        args            config_file=/etc/pnp4nagios/npcd.cfg
    }
```

This will set the module to write all perfdata suffixed with a timestamp into the perfdata spooldir defined in npcd.cfg - and then again, start NPCD in order to process the performance data into rrdtool.

**This is extremly handy to keep inGraph side by side with PNP for graphing data (and develop and test stuff).**


## Integration into Icinga

### Classic-UI

[PNP4Nagios in Classic-UI](03_03_php4nagios_in_classicui.md)


### Icinga-web

[Setting up PNP4Nagios with Icinga-Web](03_04_setting_up_pnp4nagios_with_icinga-web.md)

## Run

```
# /etc/init.d/npcd start
# /etc/init.d/icinga reload
```

### Check for perfdata

```
# ls -ltr /usr/local/pnp4nagios/var/perfdata/localhost
```

### Verify Web
Open PNP in Classic UI, and check for errors on the Kohana default page. Then remove install.php

```
# rm /usr/local/pnp4nagios/share/install.php
```
