# check_snmp

## Generic
Provided with nagios-plugins.

get an idea if the host is reponding:

```
$ snmpwalk $hostname -v $version -c $community -On .1.3.6.1.2.1.1.1.0
```

where $version is "1" or "2c"

## Net-SNMP Perl
* A fixed version of the Manubulon plugins can be found in my github repo here - https://github.com/dnsmichi/manubulon-snmp
* Further patches are collected there as well. For questions, head over to http://www.monitoring-portal.org

### Can't locate utils.pm in @INC
Perl plugins normally need utils.pm provided in plugin path which is not in the perl lib path by default. The easier way is to adapt the plugin to use a proper plugin path.
Icon

**utils.pm** is provided with the nagios-plugins package!

```
# apt-get install nagios-plugins
```

```
# sed -i 's/\/usr\/local\/nagios\/libexec/\/usr\/lib\/nagios\/plugins/g' check_snmp_storage.pl
```

### Argument "v6.0.1" isn't numeric in numeric lt (<)
This sources from old plugins getting a changed version string in libnet-snmp-perl. This can be fixed by replacing all occurences of the integer less than equal by the string compare 'lt'.

```
# sed -i 's/Net::SNMP->VERSION < 4/Net::SNMP->VERSION lt 4/g' check_snmp_storage.pl
```

### Filesystem
http://nagios.manubulon.com/snmp_storage.html

```
# apt-get install libnet-snmp-perl libnagios-plugin-perl nagios-plugins
# cd /usr/lib/nagios/plugins
# wget http://nagios.manubulon.com/check_snmp_storage.pl
# chmod +x check_snmp_storage.pl
# sed -i 's/Net::SNMP->VERSION < 4/Net::SNMP->VERSION lt 4/g' check_snmp_storage.pl
```

In order to run it, make sure to provide
* hostname (-H)
* community (-C)
* protocol (-2)
* filesystem regex (-m)
* warning/critical threshold (-w, -c)
* performance data output (-f)

**Info:** Performance data can be used straight with [PNP](../installation-guides/03_06_setting_up_pnp_with_icinga_on_debian.md) as graphing addon

```
# perl check_snmp_storage.pl -H $hostname -C $community -2 -m $filesystem-regex -w $warninglevel -c $criticallevel -f
```

### Interface
http://nagios.manubulon.com/snmp_int.html

```
# apt-get install libnet-snmp-perl libnagios-plugin-perl nagios-plugins
# cd /usr/lib/nagios/plugins
# wget http://nagios.manubulon.com/check_snmp_int.pl
# chmod +x check_snmp_int.pl

# sed -i 's/Net::SNMP->VERSION < 4/Net::SNMP->VERSION lt 4/g' check_snmp_int.pl
```

In order to run it, make sure to provide
* hostname (-H)
* community (-C)
* protocol (-2)
* interface (-n)
* warning/critical threshold (-w, -c)
* input/ouput bandwidth (-k, 2 w/c levels)
*  performance data output (-f)

**Info:** Performance data can be used straight with [PNP](../installation-guides/03_06_setting_up_pnp_with_icinga_on_debian.md) as graphing addon

```
# perl check_snmp_int.pl -H $hostname -C $community -2 -n $interface -w $warninglevel -c $criticallevel -f
e.g. bandwidth
# perl check_snmp_int.pl -H $hostname -C $community -2 -n $interface -k -w 200,400 -c 0,600 -f
```

### Memory
Since there have been numerous complaints that the check_mk hr_mem check does not report the correct values for memory and swap, you can simply attach an active check via check_snmp_mem.pl which does the right thing for years now.
http://nagios.manubulon.com/snmp_mem.html

```
# apt-get install libnet-snmp-perl libnagios-plugin-perl nagios-plugins
# cd /usr/lib/nagios/plugins
# wget http://nagios.manubulon.com/check_snmp_mem.pl
# chmod +x check_snmp_mem.pl

# sed -i 's/Net::SNMP->VERSION < 4/Net::SNMP->VERSION lt 4/g' check_snmp_mem.pl
```

In order to run it, make sure to provide
* hostname (-H)
* community (-C)
* protocol (-2)
* warning/critical thresholds "memory, swap" (-w, -c)
* performance data output (-f)

**Info:** Performance data can be used straight with [PNP](../installation-guides/03_06_setting_up_pnp_with_icinga_on_debian.md) as graphing addon

```
# perl check_snmp_mem.pl -H $hostname -C $community -2 -w $ram_warninglevel,$swap_warninglevel -c $ram_criticallevel,$swap_criticallevel -f
e.g. 80% mem crit, 20% swap
# perl check_snmp_mem.pl -H $hostname -C $community -2 -w 70,10 -c 80,20 -f
```

#### Config
Example command and service integration.

```
define  command {
        command_name                    check_snmp_mem
        command_line                    $USER1$/check_snmp_mem.pl -H $HOSTADDRESS$ -C $ARG1$ -w $ARG2$ -c $ARG3$ -f
}

define service {
        service_description             memory
        host_name                       yourhostname
        check_command                   check_snmp_mem!community!95,20!97,30
        use                             generic-service
}
```

### Hints
On Windows, you might experience null-terminated strings on network interfaces. See here for an example and solution (in german).

http://www.monitoring-portal.org/wbb/index.php?page=Thread&threadID=26504
