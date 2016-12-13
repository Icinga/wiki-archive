# Enable RRDCached in PNP

## General
http://oss.oetiker.ch/rrdtool/doc/rrdcached.en.html

> rrdcached is a daemon that receives updates to existing RRD files, accumulates them and, if enough have been received or a defined time has passed, writes the updates to the RRD file. A flush command may be used to force writing of values to disk, so that graphing facilities and similar can work with up-to-date data.

**General purpose - decrease I/O, increase performance.**

## Software

### Debian based

```
# apt-get install rrdcached
# update-rc.d rrdcached defaults
```

### RHEL based
Add repoforge to your repository list - http://repoforge.org/use/

```
# yum -y install rrdtool
# service rrdcached start
# chkconfig rrdcached on
```

## Config
Restart rrdcached after changing the configs.

### Debian based

```
# mkdir -p /var/cache/rrdcached

# vim /etc/default/rrdcached

OPTS="-w 1800 -z 1800 -j /var/cache/rrdcached -s icinga -m 0660 -l unix:/var/run/rrdcached.sock"


# service rrdcached restart
```

**For debian packages, use "-s nagios" and put the user www-data into the nagios group (e.g. by editing /etc/group)**

### RHEL based

```
# vim /etc/sysconfig/rrdcached

# Settings for rrdcached
OPTIONS="-w 1800 -z 1800 -p /var/rrdtool/rrdcached/rrdcached.pid -j /var/tmp -s icinga -m 0666 -l unix:/var/run/rrdcached.sock -t 8 "
RRDC_USER=icinga
```

### RHEL 7 / CentOS 7 based
Basing on the suggested solution from here: https://www.andrisoft.com/support/portal/kb/article/speeding-up-ip-graphs-updates-using-rrdcached#redhat7
Verified for rrdtool v1.4.8

#### Setup
Create the rrdcached service file

```
nano /usr/lib/systemd/system/rrdcached.service
```

Type the following lines within:

```
[Unit]
Description=RRDtool cache
#daemonAfter=network.target

[Service]
Type=forking
ExecStart=/usr/bin/rrdcached -s icinga -m 774 -l unix:/var/rrdtool/rrdcached/rrdcached.socket -w 3600 -f 7200 -p /var/rrdtool/rrdcached/rrdcached.pid -j /var/rrdtool/rrdcached/

[Install]
WantedBy=multi-user.target
```

**Take care that the user "apache" is also part of group "icinga"**

Save and Exit.

Create the rrdcached directory

```
mkdir -p /var/rrdtool/rrdcached/
chown root:icinga /var/rrdtool/rrdcached/
```

Start the rrdcached service

```
systemctl enable rrdcached
systemctl start rrdcached
```

Now pnp4nagios should show the graphics

**Info:** Without any further modifications the graphics will be updates each hour - be patiened

**Info:** Keep in mind that the location of the rrdcached dir differs from the location mentioned in the following chapter ( /var/rrdtool/rrdcached/rrdcached.socket instead of /var/run/ )

#### Troubleshoting
To verify function you can check the status of the process itself

```
while true; do clear ; echo STATS | socat - /var/rrdtool/rrdcached/rrdcached.socket; sleep 1; done
```

this will show you the queues of the rrdcached

```
9 Statistics follow
QueueLength: 0
UpdatesReceived: 40508
FlushesReceived: 0
UpdatesWritten: 493
DataSetsWritten: 31273
TreeNodesNumber: 29
TreeDepth: 6
JournalBytes: 2970340
JournalRotate: 8
```

**Info:** If the statistic keeps on showing no progress, please check the logfiles of process-perfdata and journalctl for further hints


## PNP Config
It might be possible, that the apache user needs to be put into icinga group.


* process_perfdata.cfg

```
RRD_DAEMON_OPTS = unix:/var/run/rrdcached.sock
```

* config.php

```
$conf['RRD_DAEMON_OPTS'] = 'unix:/var/run/rrdcached.sock';
```

* Optional

the "summery.cgi" and "avail.cgi" links will not work if your cgi-bin files are in the icinga and not nagios3 folder.

To fix, you should change the cgi-path from "/nagios3" to "/icinga" in the config.php like this: `$conf['nagios_base'] = "/cgi-bin/icinga";`


Restart
* rrdcached
* apache webserver
* to complete the change.

## Check Plugin
[check_rrdcached ](https://github.com/formorer/nagios-plugins/blob/master/check_rrdcached/check_rrdcached)
