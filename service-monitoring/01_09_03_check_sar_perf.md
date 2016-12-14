# check_sar_perf

## General
This nifty plugin works in combination with NRPE.

https://github.com/nickanderson/check-sar-perf

**Attention:** sar uses the locale time formatting, which may not work exactly like the plugin parser when called through NRPE.
a properly fixed version using LC_TIME="POSIX" sar ... can be found at my github repo: https://github.com/dnsmichi/icinga-plugins/blob/master/scripts/check_sar_perf.py

## Setup

```
# apt-get install sysstat
# cd /usr/lib/nagios/plugins
# wget https://raw.github.com/dnsmichi/icinga-plugins/master/scripts/check_sar_perf.py -O check_sar_perf.py
# chmod +x check_sar_perf.py
```

### Profiles

```
    # Profiles may need to be modified for different versions of the sysstat package
    # This would be a good candidate for a config file
    myOpts = {}
    myOpts['pagestat'] = 'sar -B 1 1'
    myOpts['cpu'] = 'sar 1 1'
    myOpts['memory_util'] = 'sar -r 1 1'
    myOpts['memory_stat'] = 'sar -R 1 1'
    myOpts['io_transfer'] = 'sar -b 1 1'
    myOpts['queueln_load'] = 'sar -q 1 1'
    myOpts['swap_util'] = 'sar -S 1 1'
    myOpts['swap_stat'] = 'sar -W 1 1'
    myOpts['task'] = 'sar -w 1 1'
    myOpts['kernel'] = 'sar -v 1 1'
    myOpts['disk'] = 'sar -d -p 1 1'
```

## Configuration

### NRPE

```
# vim /etc/nagios/nrpe.d/icinga.cfg

# sar
command[check_sar_pagestat]=/usr/lib/nagios/plugins/check_sar_perf.py pagestat
command[check_sar_cpu]=/usr/lib/nagios/plugins/check_sar_perf.py cpu
command[check_sar_memory_util]=/usr/lib/nagios/plugins/check_sar_perf.py memory_util
command[check_sar_memory_stat]=/usr/lib/nagios/plugins/check_sar_perf.py memory_stat
command[check_sar_io_transfer]=/usr/lib/nagios/plugins/check_sar_perf.py io_transfer
command[check_sar_queueln_load]=/usr/lib/nagios/plugins/check_sar_perf.py queueln_load
command[check_sar_swap_util]=/usr/lib/nagios/plugins/check_sar_perf.py swap_util
command[check_sar_swap_stat]=/usr/lib/nagios/plugins/check_sar_perf.py swap_stat
command[check_sar_task]=/usr/lib/nagios/plugins/check_sar_perf.py task
command[check_sar_kernel]=/usr/lib/nagios/plugins/check_sar_perf.py kernel
command[check_sar_disk]=/usr/lib/nagios/plugins/check_sar_perf.py disk <yourdisk>

# service xinetd restart
```

### Icinga
As checkmk checks, e.g. using the check_nrpe_command command definition.

```
## sar
  ( ( "check_nrpe_command!check_sar_pagestat", "SAR Paging Statistics", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_cpu", "SAR CPU Statistics", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_memory_util", "SAR Memory Utilization Statistics", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_memory_stat", "SAR Memory Statistics", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_io_transfer", "SAR I/O Statistics", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_queueln_load", "SAR Queue Length/Load Avg", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_swap_util", "SAR Swap Utlization Statistics", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_swap_stat", "SAR Swapping Statistics", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_task", "SAR Task Creation/System Switching", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_kernel", "SAR Kernel Tables Status", True ) , [ "host-cluster" ], ALL_HOSTS ),
  ( ( "check_nrpe_command!check_sar_disk", "SAR Block Device Activity", True ) , [ "host-cluster" ], ALL_HOSTS ),
```
