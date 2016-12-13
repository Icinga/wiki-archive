# Nagios Plugins

**Info:** Plugins are required for defining and running commands with [Icinga Core](../installation-guides/01_00_setting_up_icinga_with_idoutils.md) and Icinga 2

## Setup

### Debian

```
# apt-get install nagios-plugins
```

### RHEL
Using repoforge.org repository

```
# yum install nagios-plugins
```

### Source
Extract the Nagios plugins source code tarball.

```
# cd /usr/src
# tar xvzf nagios-plugins-1.4.16.tar.gz
# cd nagios-plugins-1.4.16
```

Compile and install the plugins by changing install directory to /usr/local/icinga (default prefix for sources, for packages /usr/lib64/nagios/plugins (RHEL) or /usr/lib/nagios/plugins (Debian)).

```
# ./configure --prefix=/usr/local/icinga \
    --with-nagios-user=icinga --with-nagios-group=icinga
# make
# make install
```

**Info:** If there is a compilation error with 'gets undeclared' please read [here](http://www.monitoring-portal.org/wbb/index.php?page=Thread&threadID=28984).

## Advanced Topics
More on the official website located at https://www.monitoring-plugins.org
