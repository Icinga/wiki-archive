#Setting up Icinga with IDOUtils on Debian

**These guides were written targetting Icinga 1.x.**

**use packages prior source install for easier upgrading**

## General
Package paths may differ! Check [Package Specific Locations]()
What will you get once Icinga with IDOUtils using this guide is installed?

* Icinga Core
* Icinga Classic UI (the "CGIs")
* Icinga IDOUtils
* Mysql or Postgresql as database
* Icinga Docs

If you came here looking for **Icinga Web Setup**, please check: [Setting up Icinga Web on Debian]().
## Offical
Icinga is already in Debian package repositories but the stable versions are rather old.
You are advised to add Debian backports to your repository list and look out for current packages.
sync with the Debian packages.


Don't forget to install the **Nagios Plugins** as well! :-)

## README
Provided with your install, see below. Current version on GIT

[Icinga README](http://anonscm.debian.org/gitweb/?p=pkg-nagios/icinga.git;a=blob;f=debian/README.Debian;hb=HEAD)

[IDOUtils Readme](http://anonscm.debian.org/gitweb/?p=pkg-nagios/icinga.git;a=blob;f=debian/icinga-idoutils.README.Debian;hb=HEAD)
## Packages

If **Stable** is too old, chose newer package from whether **Backports** or the **Builds from debmon.org**

The Debian packages will provide an automated database setup, which you will be asked upon install.

### Stable Install

```
# apt-get install mysql-server libdbd-mysql mysql-client
or
# apt-get install postgresql libdbd-pgsql postgresql-client
```
```
# apt-get install icinga icinga-cgi icinga-core icinga-doc icinga-idoutils
```

### Backports Install
[http://backports.debian.org/Instructions/](http://backports.debian.org/Instructions/)

[http://packages.debian.org/squeeze-backports/icinga](http://packages.debian.org/squeeze-backports/icinga)

```
# echo "deb http://backports.debian.org/debian-backports squeeze-backports main" >> /etc/apt/sources.list.d/squeeze-backports.list
# apt-get update
```

```
# apt-get install mysql-server libdbd-mysql mysql-client
or
# apt-get install postgresql libdbd-pgsql postgresql-client
```

```
# apt-get -t squeeze-backports install icinga icinga-cgi icinga-core icinga-doc icinga-idoutils
```

### debmon Install

[http://debmon.org/instructions](http://debmon.org/instructions)

```
# echo "deb http://backports.debian.org/debian-backports squeeze-backports main" >> /etc/apt/sources.list.d/squeeze-backports.list
# echo "deb http://debmon.org/debmon debmon-squeeze main" >> /etc/apt/sources.list.d/debmon.list
# apt-get update
```

```
# apt-get install mysql-server libdbd-mysql mysql-client
or
# apt-get install postgresql libdbd-pgsql postgresql-client
```

```
# apt-get install icinga icinga-cgi icinga-core icinga-doc icinga-idoutils
```

### Plugins
	apt-get install nagios-plugins
## Configuration

Follow `/usr/share/doc/icinga/README.Debian` and `/usr/share/doc/icinga-idoutils/README.Debian` **closely**!!!

### Enable ido2db Daemon
```
# vim /etc/default/icinga
IDO2DB=yes
# service ido2db start

```
In order to check the enabled status on ido2db startup, use

`# sh -x /etc/init.d/ido2db start | grep IDO2DB`

### Enable idomod module

Check whether this has already been done in `/etc/icinga/modules/idoutils.cfg`. If not, copy the sample config over and restart Icinga to load the module. (Tip - if icinga.cfg does not contain `cfg_dir=/etc/icinga/modules` the config won't be included!).

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

`# htpasswd /etc/icinga/htpasswd.users youradmin`


