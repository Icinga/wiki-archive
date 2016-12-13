# check_iostat

The original can be found [here](http://exchange.nagios.org/directory/Plugins/Uncategorized/Operating-Systems/Linux/check_iostat--2D-I-2FO-statistics/details) but got patched a bit further [here](http://www.pn-it.com/linux-ubuntu/nagios-festplatten-mit-check_iostat-uberwachen/) as it needed MB instead of KB. Since this does not work flawlessly, a fixed version is [attached](attachments/check_iostat).

**Info:** Added my version to github, makes changes easier: https://github.com/dnsmichi/icinga-plugins

Decide which version you want to use, the config is all the same. Download, put into your plugins dir, and set executable flag.

```
$ ./check_iostat -d sda -w 100,100,100 -c 200,200,200
OK - I/O stats tps=1.00 MB_read/s=0.00 MB_written/s=0.00 | 'tps'=1.00; 'MB_read/s'=0.00; 'MB_written/s'=0.00;
```

```
define command{
        command_name    check_iostat
        command_line    $USER1$/check_iostat -d $ARG1$ -w $ARG2$ -c $ARG3$
        }

define service{
        use                             local-service
        host_name                       localhost
        service_description             IO-Stats /dev/sda
        check_command                   check_iostat!sda!100,100,100!200,200,200
}
```

Alternative Version here: https://build.opensuse.org/package/files?package=nagios-plugins-sar-perf&project=server%3Amonitoring
