#Setting up Icinga with IDOUtils on Ubuntu


These guides were written targetting  **Icinga 1.x.**


use packages prior source install for easier upgrading


Package paths may differ! Check [Package Specific Locations]()

Don't forget to install the **Nagios Plugins** as well! :-)

##General

What will you get once Icinga with IDOUtils using this guide is installed?

* Icinga Core
* Icinga Classic UI (the "CGIs")
* Icinga IDOUtils
	* Mysql or Postgresql as database
* Icinga Docs

If you came here looking for **Icinga Web Setup**, please check: Setting up [Icinga Web on Ubuntu]().

##README
This is named **README.Debian** as the packages where original imported from Debian!



[Icinga README](http://anonscm.debian.org/gitweb/?p=pkg-nagios/icinga.git;a=blob;f=debian/README.Debian;hb=HEAD)

[IDOUtils Readme](http://anonscm.debian.org/gitweb/?p=pkg-nagios/icinga.git;a=blob;f=debian/icinga-idoutils.README.Debian;hb=HEAD)

##Packages
Chose whether to use **Official** or the Debian packager's **PPA**. Note: Official may be outdated, while the other may provide newer packages.


There's a dedicated dbconfig-common run when installing the packages. Provide Postgres and Mysql password details correctly in order to let the IDOUtils database setup happen automagically, as well as the ido2db.cfg configuration stuff.


###Official

The official packages can be found here: [launchpad.net Icinga](https://launchpad.net/ubuntu/+source/icinga)

```
# apt-get install icinga icinga-doc icinga-idoutils mysql-server libdbd-mysql mysql-client
or
# apt-get install icinga icinga-doc icinga-idoutils postgresql libdbd-pgsql postgresql-client
```

The database packages are not needed if you don't plan to use idotuils, or if your database is on a different host (then only the dbd provider is needed).

###PPA

The Debian Maintainer also provides Packages for Ubuntu via his Icinga PPA they are kept in sync with Icinga releases and the Debian packages. If you want up2date Icinga packages, this is the way to go.

```
# add-apt-repository ppa:formorer/icinga
# apt-get update
# apt-get install icinga icinga-doc icinga-idoutils mysql-server libdbd-mysql mysql-client
or
# apt-get install icinga icinga-doc icinga-idoutils postgresql libdbd-pgsql postgresql-client
```


The database packages are not needed if you don't plan to use idotuils, or if your database is on a different host (then only the dbd provider is needed).


###Plugins

	# apt-get install nagios-plugins

##Configuration

**Follow** /usr/share/doc/icinga/README.Debian and /usr/share/doc/icinga-idoutils/README.Debian **closely**!!!
Enable ido2db Daemon

### Enable ido2db Daemon

```
# vim /etc/default/icinga
IDO2DB=yes
 
# service ido2db start
```
In order to check the enabled status on ido2db startup, use
	
	# sh -x /etc/init.d/ido2db start | grep IDO2DB
 

### Enable idomod module



Check whether this has already been done in /etc/icinga/modules/idoutils.cfg. If not, copy the sample config over and restart Icinga to load the module. (Tip - if icinga.cfg does not contain `cfg_dir=/etc/icinga/modules` the config won't be included!).

```
# cp /usr/share/doc/icinga-idoutils/examples/idoutils.cfg-sample /etc/icinga/modules/idoutils.cfg
# service icinga restart
```

Check your icinga log and/or syslog to verify that Icinga with idomod has been started, as well as idomod has successfully connected to ido2db.

### Enable external commands

If you were not asked during package install, do it manually.

```
# vim /etc/icinga/icinga.cfg
check_external_commands=1
 
# service icinga stop
# dpkg-statoverride --update --add nagios www-data 2710 /var/lib/icinga/rw
# dpkg-statoverride --update --add nagios nagios 751 /var/lib/icinga
# service icinga start
```

### Classic UI Authentication

The authorization is stored within  /etc/icinga/htpasswd.users - new users can be added with the following command
	
	# htpasswd /etc/icinga/htpasswd.users youradmin
 