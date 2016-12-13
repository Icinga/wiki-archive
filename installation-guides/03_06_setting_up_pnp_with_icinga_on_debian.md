# Setting up PNP with Icinga on Debian

**Info:** As usual, going the package way is the easiest one - if you prefer to install from source, see ?Setting up PNP with Icinga. In this guide, we will be using the Debian backports to install the latest stable pnp4nagios packages.

## Requirements
* Icinga running
* Icinga Classic UI or Web running

## Setup

```
# apt-get update
```

```
# apt-get install --no-install-recommends pnp4nagios
```

This is a meta package and will install pnp4nagios-bin and pnp4nagios-web. Without --no-install-recommends it will also install Icinga if not installed.


**Info:** Make sure to [Enable RRDCached in PNP](03_01_enable_rrdcached_in_pnp.md)

## Config
We will install PNP with "Bulk mode with NPCD", the package readme will use a different mode, called "bulk mode with npcdmod and NPCD". Decide yourself then.

```
# cd /usr/share/doc/pnp4nagios
# less README.debian
```

### Perfdata in Icinga
Add to **icinga.cfg**

```
# performance data
process_performance_data=1

service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$

host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$

service_perfdata_file_mode=a
host_perfdata_file_mode=a

service_perfdata_file_processing_interval=30
host_perfdata_file_processing_interval=30

service_perfdata_file=/var/spool/pnp4nagios/nagios/service-perfdata
host_perfdata_file=/var/spool/pnp4nagios/nagios/host-perfdata

service_perfdata_file_processing_command=pnp-bulknpcd-service
host_perfdata_file_processing_command=pnp-bulknpcd-host

#define command{
#        command_name    pnp-bulknpcd-service
#        command_line    /bin/mv /var/spool/pnp4nagios/nagios/service-perfdata /var/spool/pnp4nagios/npcd/service-perfdata.$TIMET$
#}

#define command{
#        command_name    pnp-bulknpcd-host
#        command_line    /bin/mv /var/spool/pnp4nagios/nagios/host-perfdata /var/spool/pnp4nagios/npcd/host-perfdata.$TIMET$
#}
```

Add to your **commands.cfg** (or whatever is included in your icinga.cfg)

```
# pnp4nagios
define command{
        command_name    pnp-bulknpcd-service
        command_line    /bin/mv /var/spool/pnp4nagios/nagios/service-perfdata /var/spool/pnp4nagios/npcd/service-perfdata.$TIMET$
}

define command{
        command_name    pnp-bulknpcd-host
        command_line    /bin/mv /var/spool/pnp4nagios/nagios/host-perfdata /var/spool/pnp4nagios/npcd/host-perfdata.$TIMET$
}
```

### Web Config
**Attention:** There's a bug in the packages targetting the wrong htpasswd for basic auth.

```
# vim /etc/apache2/conf.d/pnp4nagios.conf

        AuthName "Icinga Access"
        AuthType Basic
        AuthUserFile /etc/icinga/htpasswd.users
```

after that, reload the webserver.

```
# service apache2 reload
```

## Run
Enable NPCD first and then start it.

```
# vim /etc/default/npcd
Run="yes"

# service npcd start
```

## Integration into Icinga

### Classic UI

```
# vim /etc/icinga/objects/pnptemplate.cfg

define host {
        name       pnp-hst
        register   0
        action_url /pnp4nagios/graph?host=$HOSTNAME$' class='tips' rel='/pnp4nagios/popup?host=$HOSTNAME$&srv=_HOST_
}

define service {
        name       pnp-svc
        register   0
        action_url /pnp4nagios/graph?host=$HOSTNAME$&srv=$SERVICEDESC$' class='tips' rel='/pnp4nagios/popup?host=$HOSTNAME$&srv=$SERVICEDESC$
}
```

and use those templates in your host and service templates, e.g.

```
define host{
        name                            generic-host
        use                             pnp-hst
...

define service{
        name                            generic-service
        use                             pnp-svc
...
```

Afterwards, copy the ssi file with the jquery popup code

```
# cp /usr/share/doc/pnp4nagios/examples/ssi/status-header.ssi /usr/share/icinga/htdocs/ssi/status-header.ssi
# chmod 644 /usr/share/icinga/htdocs/ssi/status-header.ssi
```

and make sure to reload Icinga to recognize the new config.

```
# service icinga reload
```

### Icinga Web
Depends how you installed it. So the first task will be to check onto the prefix - default is **/usr/local/icinga-web** which will be used in this section.

To leave grid templates untouched we integrated XML extension to customize grid templates with simple snippets. PNP integration is made with these extension to be upgrade safe.

**Warning:**
These samples are provided with the source tarball (see Setting up Icinga Web on Debian) - the install commands require you to get there like

```
cd /path/to/your/icinga-web-src
```

```
cat contrib/PNP_Integration/INSTALL
************************
* INSTALLATION
************************

To install this addon, simply copy both xml files under templateExtensions to
your icinga-webs app/modules/Cronks/data/xml/extensions folder and clear the app/cache/CronkTemplates folder

To remove it, just delete the extension files and clear the cache folder again
```

```
# cp contrib/PNP_Integration/templateExtensions/*.xml /usr/local/icinga-web/app/modules/Cronks/data/xml/extensions/
# /usr/local/icinga-web/bin/clearcache.sh
```
