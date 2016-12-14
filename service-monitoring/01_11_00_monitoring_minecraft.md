# Monitoring Minecraft

## Minecraft
* http://www.minecraft.net/
* http://www.debianroot.de/server/minecraft-server-erstellen-auf-einem-debian-server-1297.html

## Checks

http://erschaffe.de/2011/04/06/three-must-have-nagios-services-for-your-minecraft-server/

Use whatever transport you like most - NRPE, ssh, snmp, etc. Described configs are NRPE specific.

Make sure Icinga knows this command.

```
define  command {
        command_name    check_nrpe_noargs
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -t 10 -c $ARG1$
}
```

### Default
Make sure to monitor the default system, like
* CPU Utilization/Load
* Memory/Swap
* Filesystems
* etc

### Listen
**nrpe.cfg**

```
command[minecraft_listen]=/usr/lib/nagios/plugins/check_tcp -H your.minecraft.host -p 25565 -w 5 -c 10
```

**icinga**

as check_command, use

```
check_command                 check_nrpe_noargs!minecraft_listen
```

### World size
Download it onto the minecraft server to /usr/lib/nagios/plugins (or wherever you put plugins) and make it executable

http://exchange.nagios.org/directory/Plugins/Uncategorized/Operating-Systems/Linux/CheckDirSize/details

**nrpe.cfg**

```
command[minecraft_worldsize]=/usr/lib/nagios/plugins/check_dirsize11.sh -d /home/minecraft/minecraft/world -w 8000000 -c 10000000 -f
```

**icinga**

as check_command, use

```
check_command                 check_nrpe_noargs!minecraft_worldsize
```

### Logs
Use [check_logfiles](01_10_01_check_logfiles.md) with special patterns in a config file and pass it to the command --config /omd/sites/nagios/check_logfiles.cfg


**nrpe.cfg**

```
command[minecraft_logs]=/usr/lib/nagios/plugins/check_logfiles --configfile /etc/nagios/check_logfiles.cfg
```

**check_logfiles.cfg**

```
@searches = (
  {
    tag => 'mc_server.log',
    logfile => '/home/minecraft/minecraft/server.log',
    criticalpatterns => 'SEVERE',
    warningpatterns => 'WARNING',
    options => 'sticky=600, perfdata'
  },
 {
    tag => 'mc_service.log',
    logfile => '/home/minecraft/minecraft/service.log',
    criticalpatterns => 'SEVERE',
    warningpatterns => 'WARNING',
    options => 'sticky=600, perfdata'
  },
);
```

**icinga**

as check_command, use

```
check_command                 check_nrpe_noargs!minecraft_logs
```

### Online Players
http://erschaffe.de/2011/04/05/howto-monitor-online-players-in-nagios-with-fancy-graphs/

This requires check_mysql_health and dependencies to be installed on the Minecraft server. If you have PNP installed, provided perfdata can generate fancy graphs as well.

**nrpe.cfg**

```
command[minecraft_players]=/usr/lib/nagios/plugins/check_mysql_health --hostname localhost --username minecraft
--password <pass> --mode sql --name "SELECT count(*) FROM minecraft.netstats WHERE logged=1;"
--critical 80 --warning 50 --name2 "players"
```

**icinga**

as check_command, use

```
check_command                 check_nrpe_noargs!minecraft_players
```
