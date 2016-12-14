# check_mathlm

**Info:** on questions, get in touch with the author!

http://cf.lehigh.edu/~kbe2/check_mathlm

```
#!/usr/bin/perl
# plugin to check Mathematica's license manager. requires their 'monotirlm' binary...
# I have no idea where you can get this. Ask whoever set up the license server.
#
# If you improve on this, please send me a copy: keith@lehigh.edu
#
# Let's call this v0.1, 4/2012

use Nagios::Plugin;

$monlm = "/usr/local/bin/monitorlm";

$np = Nagios::Plugin->new(
      usage => "Usage: %s [-v|--verbose] -H|--host <host> [-t|--timeout <timeout>]",
      );

$np->add_arg( spec => 'host|H=s', help => "--host|-H\n   host name", required => 1);

$np->getopts;

alarm $np->opts->timeout;
$host = $np->opts->host;
$verbose = $np->opts->verbose;

open MON, "$monlm $host 2>&1 |";

%lic = {};

while (<MON>) {
    if ($verbose) {
    print $_;
    }
    #for "Could not contact MathLM server...", "Could not find a MathLM server", etc.
    if ($_ =~ /^Could not/) {
    #BAIL OUT, BAIL OUT
    $np->nagios_exit(CRITICAL, $_);
    }

    #distgusting regex to pluck out the license counts
    if ($_ =~ /^((?:Sub )?Math(?:ematica|Kernel))\s*\w\s*(\d+)\s*(\d+)/) {
    $lic->{$1}->{auth} = $3;
    $lic->{$1}->{used} = $2;
    }

}

close MON;

$status = OK;

#this loop could/should probably use the add_message() functionality, but it's marked "experimental"...
while( ($prod, $count) = each %$lic ) {
    if ($verbose) {
    print "found: $prod, licenses: $count->{'used'} / $count->{'auth'}\n";
    }
    if ($count->{'used'} >= $count->{'auth'}) {
    $status = WARNING;
    $statmsg .= "All $prod licenses in use. ";
    }
}

if ($status != OK) {
    $np->nagios_exit($status, $statmsg);
}

#only return OK if we actually found some licenses...
if ( keys( %$lic ) ) {
    $np->nagios_exit(OK, "License server okay.");
}

#otherwise, I have no idea what the status is
$np->nagios_exit(UNKNOWN, "License server responded, but I didn't find any usage numbers?");
```
