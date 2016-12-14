# Clustered Checks with Icinga and check_multi

## Setup

### NRPE
Setup NRPE like described here: Setting up NRPE with Icinga

### check_multi
Matthias Flacke created [check_multi](http://my-plugin.de/wiki/projects/check_multi/start) which basically supports multiple checks within a single check command.

We will use that possbility in order to execute 1..n checks and calculate the overall returned state and output (and perfdata in case).

#### Installation
Download check_multi, put it into the location of $USER1$ defined in resource.cfg

```
$ git clone git://github.com/flackem/check_multi ; cd check_multi
$ ./configure --with-nagios-name=icinga --with-nagios-user=icinga --with-nagios-group=icinga
$ make all
$ sudo make install
```

**Info:** If you want to use a dedicated plugins path, add **--libexecdir=/usr/lib/nagios/plugins** to configure.

**Info:** Using Debian packages it's more easy - having backports enabled from Setting up [Icinga with IDOUtils on Debian](../installation-guides/01_01_setting_up_icinga_with_idoutils_on_debian.md)

`# apt-get install nagios-plugin-check-multi`

#### Configuration
**Note:** Paths may differ depending on your installation! The provided paths use the default prefix running configure without any prefix param.

##### Checks

On the remote nodes you will need
* plugins e.g. check_ping
* command definition in nrpe.cfg
  * `command[check_ping]=/usr/lib/nagios/plugins/check_ping -H $ARG1$ -w $ARG2$ -c $ARG3$ -p $ARG4$ -4 -t $ARG5$`
* a running NRPE server accepting connections from the Icinga master
  * dont_blame_nrpe=1 (having command arguments enabled)

On the Icinga master you will need

* NRPE client
* check_multi
* plugins for local checks

###### check_multi command definition

Define check_multi check commands in order to use them afterwards. It is necessary to tell check_multi which macros should be used and piped into. We are also using a default path for all expected `*.cmd` files ($ARG1$).

**Info:** Debian packages: ***/usr/lib/nagios/plugins/check_multi***

```
define command {
  command_name         check_multi
  command_line         /usr/local/icinga/libexec/check_multi -l /usr/local/icinga/libexec -s HOSTADDRESS=$HOSTADDRESS$ -s HOSTADDRESS6=$HOSTADDRESS6$ -f /usr/local/icinga/etc/$ARG1$ $ARG2$ $ARG3$ $ARG4$
}
```

###### service definition
The clustered check definition will look combine the check_multi command with the desired checks. So this can be re-used just by changing the check_command args in various other clustered checks - if you happen to define more check_multi cmd files

```
define  service {
        use                             generic-service
        host_name                       localhost
        service_description             PING-PROBES
        check_command                   check_multi!check_multi_node_ping.cmd
}
```

Note: You can pass more macros to check_multi using the '-s' flag. Read their docs for further instructions on that.


###### check_multi command file
Define the check commands for pinging local and remote nodes and the conditions how the return state will be (this is up to you!). Replace X.remote.node with the remote node running NRPE server.

```
# vim /usr/local/icinga/etc/check_multi_node_ping.cmd

# If all Locations show red, the host id dead. (Critical > 2)
# Warn me if one location is critical or two are in the warning state

command[ localnode ]              = check_ping -4 -H $HOSTADDRESS$ -w 3000.0,80% -c 5000.0,100% -p 5
command[ remotenode1 ]            = check_nrpe -H 1.remote.node -c check_ping -a $HOSTADDRESS$ 3000.0,80% 5000.0,100% 5 10
command[ remotenode2 ]            = check_nrpe -H 2.remote.node -c check_ping -a $HOSTADDRESS$ 3000.0,80% 5000.0,100% 5 10

state   [ CRITICAL   ] = COUNT(CRITICAL) > 2
state   [ WARNING    ] = COUNT(WARNING) > 1 || COUNT(CRITICAL) > 1
state   [ UNKNOWN    ] = COUNT(UNKNOWN) > 2
```

###### Hints

* use cgi.cfg:escape_html_tags=0 to allo cgis to show the output html style
* you can re-use already existing check outputs/states using "ondemand macros" from Icingahttp://my-plugin.de/wiki/projects/check_multi/configuration/macros

## Examples

### Clustered IPv6 Check

Checking if my webhosts publicy (Google DNS @8.8.8.8) resolve into valid AAAA records (are you ready for IPv6?)


```
#
# www v6 dns resolve clustered check
#

command[ www-univie-v6 ]                  = /usr/lib/nagios/plugins/check_dig -H 8.8.8.8 -l www.univie.ac.at. -T AAAA
command[ www-aconet-v6 ]                  = /usr/lib/nagios/plugins/check_dig -H 8.8.8.8 -l www.aco.net. -T AAAA
command[ www-nicat-v6 ]                  = /usr/lib/nagios/plugins/check_dig -H 8.8.8.8 -l www.nic.at. -T AAAA

state   [ CRITICAL   ] = COUNT(CRITICAL) > 2
state   [ WARNING    ] = COUNT(WARNING) > 2 || COUNT(CRITICAL) > 1
state   [ UNKNOWN    ] = COUNT(UNKNOWN) > 1
```

### Postgresql Process Cluster Check

```
# Postgresql Cluster Proc Check

command[ postgres_1a ]         = /usr/lib/nagios/plugins/check_snmp_process.pl -H 1a.hostname -2 -C community -n postgres -F
command[ postgres_1b ]         = /usr/lib/nagios/plugins/check_snmp_process.pl -H 1b.hostname -2 -C community -n postgres -F

# case scenarios
# 1. 1a ok, 1b must have 0 procs (1xok, 1xcrit) -
# 2. 1a !ok, 1b must have >0 procs (1xcrit, 1xok)
# 3. both have >0 procs (2xok) - critical (brainsplit)
# 4. both have 0 procs (2x critical) - critical (failure)

state   [ OK         ] = COUNT(OK) == 1
state   [ CRITICAL   ] = COUNT(CRITICAL) > 1 || (COUNT(UNKNOWN) > 0 && COUNT(CRITICAL) > 0) || COUNT(OK) > 1
state   [ WARNING    ] = COUNT(WARNING) > 1
state   [ UNKNOWN    ] = COUNT(UNKNOWN) > 1
```
