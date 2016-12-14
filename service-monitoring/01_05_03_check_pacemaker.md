# check_pacemaker

## Knowhow

> crm configure show: dump current config to stdout
>
> crm configure save <file>: dump current config into file

## Checks
These output could be used for monitoring.

```
# crm_mon --help
```

### check_crm
Base Check.

http://exchange.nagios.org/directory/Plugins/Clustering-and-High-2DAvailability/Check-CRM/details

[check_crm_v0_5](attachments/check_crm_v0_5)

```
# visudo

nagios  ALL=NOPASSWD: /usr/sbin/crm_mon -1 -r -f
```

### check_pcmkactions
check failed actions

http://www.s-reimann.com/2011/06/30/nagios-check-pacemaker-failed-actions/

```
#!/bin/bash
#
# title: check_pcmkactions - check for failed pacemaker actions
# date created: Tue Jan 25 2011
# last edit: Thu Jun 30 2011
# author: Sascha Reimann
# changelog: - crm_mon, awk & grep put into variables.
# nagios returncodes:
STATE_OK=0
STATE_WARNING=1
STATE_CRITICAL=2
STATE_UNKNOWN=3
crm_mon="/usr/sbin/crm_mon"
awk="/usr/bin/awk"
grep="/bin/grep"
# check for failed actions and set $STATUS
$crm_mon --one-shot | $grep --quiet "Failed"
STATUS=$?
# generate output:
if [ ${STATUS} -eq 0 ]
then
DETAILS=$($crm_mon --one-shot | $awk '/Failed/ {f=1}f' | $grep --invert-match Failed)
COUNT=$($crm_mon --one-shot | $awk '/Failed/ {f=1}f' | $grep --invert-match --count Failed)
echo "CRITICAL: ${COUNT} failed action(s): ${DETAILS}"
exit ${STATE_CRITICAL}
elif [ ${STATUS} -eq 1 ]
then
echo "OK: no failed actions found"
exit ${STATE_OK}
else
echo "UNKNOWN: returncode ${STATUS}"
exit ${STATE_UNKNOWN}
fi
```

### check-pacemaker-config
Passive Check, via Cronjob

#### Prerequisites

#### Cachedir
`# mkdir /var/cache/pacemaker`

#### Template
Mandatory Template as master, which is diffed onto.

```
/var/cache/pacemaker/pacemaker-config.template
```

Put the current config into the template

```
# crm configure save /var/cache/pacemaker/pacemaker-config.template
```

#### Setup

* dump current pacemaker config
* diff aginst template
* if diff available, alarm
  * extract the diff, do linecount
* if diff is not possible (no dump), die with unknown

#### Code

`# vim /usr/local/bin/check-pacemaker-config`

```
#!/usr/bin/perl
#working dir
my $cachedir = '/var/cache/pacemaker';
#save this once with # crm configure save /var/cache/pacemaker/pacemaker-config.template
my $templatecfgfile = $cachedir.'/'.'pacemaker-config.template';
#current dumped config
my $cfgfile = $cachedir.'/'.'pacemaker-config';
#nsca information
my $icinga_nscahost = 'icingahost';
my $icinga_nscacfg = '/etc/send_nsca.cfg';
#icinga information
my $icinga_host = 'host';
my $icinga_svc = 'db-pacemaker-config';
my $icinga_status = 0; # OK
my $icinga_output = "OK";
my $icinga_perfdata = "lines=0";
my $icinga_longoutput = "";
my $icinga_complete_output = "";
my $lines = 0;
#check if there was a previous leftover
if(-e $cfgfile) {
        unlink($cfgfile);
}
#dump it
`/usr/sbin/crm configure save $cfgfile`;
#check if the dump happened and then diff, bail out unknown if not
if(-e $cfgfile) {
        #diff it to the teplate, e.g. gotten from git
        $diff = `diff $templatecfgfile $cfgfile`;
        #do something if the diff is there
        if ($diff ne "") {
                #print "diff is $diff";
                # FIXME do something with longoutput - current NSCA does not support multiline
                $icinga_longoutput = `echo "$diff" | egrep "^<|>"`;
                $lines = $icinga_longoutput =~ tr/\n//;
                $icinga_status = 2; # CRITICAL
                $icinga_output = "CRITICAL - config differs ($lines)!";
                $icinga_perfdata = "lines=$lines";
                # FIXME - escape for send_nsca
                #$icinga_longoutput =~ s/\R//g;
                $icinga_longoutput = "";
        }
} else {
        $icinga_status = 3; # UNKNOWN
        #print "could not diff this time";
        $icinga_output = "UNKNOWN - no diff possible";
        $icinga_perfdata = "lines=0";
        $icinga_longoutput = "";
}

$icinga_complete_output = "$icinga_output|$icinga_perfdata\n$icinga_longoutput";
`echo "$icinga_host\t$icinga_svc\t$icinga_status\t$icinga_complete_output\n" | /usr/sbin/send_nsca -H $icinga_nscahost -c $icinga_nscacfg`;
print "sending the following to send_nsca...\n";
print "$icinga_host\t$icinga_svc\t$icinga_status\t$icinga_complete_output\n";
```

`# chmod +x /usr/local/bin/check-pacemaker-config`

#### Crontab

```
# vim /etc/crontab

# db-pacemaker-config check
0,15,30,45 * * * * root perl /usr/local/bin/check-pacemaker-config >> /dev/null 2>&1
```

#### Icinga

```
define  service {
        use                             passive-reports-template
        host_name                       host
        service_description             db-pacemaker-config
        freshness_threshold             120 # 2h
        check_command                   db-pacemaker-config!No Report Received!
}
```
