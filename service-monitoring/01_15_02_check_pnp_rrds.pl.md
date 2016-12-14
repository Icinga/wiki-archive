# check_pnp_rrds.pl

Provided within PNP in order to check for stale RRDs (XMLs of PNP) which are not updated anymore.

```
./check_pnp_rrds.pl -h
Copyright (c) 2008 Joerg Linge, Pitchfork@pnp4nagios.org


check_pnp_rrds.pl 0.6.16
check_pnp_rrds.pl is used to find old or unusable RRD Files

USAGE: check_pnp_rrds.pl [OPTIONS]
  -w,--warning
       Default: 1
  -c,--critical
        Default: 10
  -a,--fileage Max XML File Age.
       Default: 7 Days
  -p,--rrdpath Path to your RRD and XML Files.
       Default: /var/lib/pnp4nagios/
  -t,--timeout Max Plugin Runtime.
       Default: 10 Seconds
  --ignore-hosts
       Regular expression to ignore a set of hosts

  --passiv-hostname=
       Nagios Hostname
  --passiv-servicedesc=
       Nagios Servicedesc
  --nagios-cmd=
       External Command File (nagios.cmd)


SUPPORT: http://www.pnp4nagios.org/pnp/
```

## Config

```
###########################################################################
# PNP
###########################################################################
define command {
        command_name    check_stale_rrds
        command_line    /usr/libexec/pnp4nagios/check_pnp_rrds.pl -p /data/pnp4nagios -a $ARG1$
}
```

```
  ( ( "check_stale_rrds!500", "PNP Stale RRDs", True ) , [ "icinga" ], ["localhost"] ),
```
