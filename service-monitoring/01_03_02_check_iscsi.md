# check_iscsi

## Server
:warning FIXME

## Initiator
Each location, which mounts storage via ISCSI. You can track down lost filesystem mounts, as well as short hickups of the service itsself.

### Session
Via NRPE, small script checking the session via iscsiadm.

```
# perl /usr/lib/nagios/plugins/check_iscsi_session 1.2.3.4
```

### Config

```
# vim /usr/lib/nagios/plugins/check_iscsi_session

#!/usr/bin/perl
use strict;
use warnings;
my $ISCSIADMIN = "/usr/bin/iscsiadm";
my $iscsi_host = $ARGV[0] || "1.2.3.4";
my @output = `$ISCSIADMIN -m session | grep $iscsi_host`;
my $session = @output;
if ($session == 0) {
        print "CRITICAL - no session found | sessions=$session\n";
        exit 2;
} else {
        my $out = join(", ", @output);
        chomp $out;
        print "OK - session found: $out | sessions=$session \n";
        exit 0;
}

# chmod +x /usr/lib/nagios/plugins/check_iscsi_session

# vim /etc/nagios/nrpe.d/icinga.cfg
command[check_iscsi_session]=/usr/lib/nagios/plugins/check_iscsi_session

# service xinetd restart

icinga

  ( ( "check_nrpe_command!check_iscsi_session", "ISCSI session", True), [ "iscsi" ], ALL_HOSTS ),

```

### Readonly Filesystems
It may happen that the ISCSI target gets lost, and the kernel remounts the filesystem readonly to prevent data loss. That's why we keep checking for that as well.

```
# vim /usr/lib/nagios/plugins/check_ro_fs

#!/usr/bin/perl
use strict;
use warnings;
my @output = `awk '\$4~/(^|,)ro(\$|,)/' /proc/mounts`;
my $count = @output;
if ($count == 0) {
        print "OK - no readonly fs found | count=$count\n";
        exit 0;
} else {
        chomp(@output);
        my $out = join(", ", @output);
        chomp $out;
        print "CRITICAL - readonly fs found: $out | count=$count \n";
        exit 2;
}

icinga

  ( ( "check_nrpe_command!check_ro_fs", "Readonly FS", True), [ "iscsi" ], ALL_HOSTS ),
```

### Process
iscsid as process, required to run. But doesn't really tell about the functionality itsself.

```
( ( "check_snmp_procs!iscsid!0!5!0!5!100!200!community", "ISCSI Proc Check", True), [ "iscsi" ], ALL_HOSTS ),
```

### Logs
If ISCSI detects an error, it will be logged - using [check_logfiles](01_10_00_log_monitoring.md) you may pattern match those, and alert accordingly.

```
Apr 12 11:51:19 host iscsid: Kernel reported iSCSI connection 1:0 error (1011) state (3)
Apr 12 11:51:31 host iscsid: connection1:0 is operational after recovery (1 attempts)
```
