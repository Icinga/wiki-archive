# check_xen

## NRPE Checks

### Clients

* https://secure.opsera.com/wsvn/wsvn/opsview/trunk/opsview-core/nagios-plugins/check_xen_number
* https://secure.opsera.com/wsvn/wsvn/opsview/trunk/opsview-core/nagios-plugins/check_xen_state

Dependencies/Software

```
# apt-get install  libdata-dump-perl nagios-nrpe-server

# cd /usr/src
# wget http://search.cpan.org/CPAN/authors/id/T/TO/TONVOON/Nagios-Plugin-0.35.tar.gz
# tar xzf Nagios-Plugin-0.35.tar.gz
# cd Nagios-Plugin-0.35/
# perl Makefile.PL ; make ; make install

# apt-get install libparams-validate-perl
```

put scripts into /usr/lib/nagios/plugins

```
/usr/lib/nagios/plugins# chmod +x check_xen_*
```

save nrpe.cfg, edit nrpe_local.cfg. call to xm requires sudo.

```
# vim /etc/nagios/nrpe_local.cfg

command[check_nrpe_daemon]=/bin/echo "NRPE OK"

command[check_xen_number]=sudo /usr/lib/nagios/nagios/check_xen_number
command[check_xen_state]=sudo /usr/lib/nagios/nagios/check_xen_state -n $ARG1$


# /etc/init.d/nagios-nrpe-server restart
# insserv nagios-nrpe-server

# ln -s /usr/sbin/xm /usr/bin/xm

# apt-get install sudo
# vim /etc/sudoers

nagios ALL=NOPASSWD:/usr/lib/nagios/plugins/check_xen_number
nagios ALL=NOPASSWD:/usr/lib/nagios/plugins/check_xen_state
```

### Icinga Master

***commands.cfg***

```
define  command {
        command_name    check_nrpe_command
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$
}

define  command {
        command_name    check_nrpe_command_args
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -t 30 -c $ARG1$ -a $ARG2$
}
```

***check_mk***

```
define  service {
...
        check_command   check_nrpe_command!check_xen_number
...
}

define  service {
...
        check_command   check_nrpe_command_args!check_xen_state!ns11h.univie.ac.at
...
}
```

### Checks

#### check_xen_number
https://secure.opsera.com/wsvn/wsvn/opsview/trunk/opsview-core/nagios-plugins/check_xen_number

```
#!/usr/bin/perl
#
# AUTHORS: Rohit Deshmukh
#   Copyright (C) 2003-2010 Opsera Limited. All rights reserved
#
#    This file is part of Opsview
#
#    Opsview is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    Opsview is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with Opsview; if not, write to the Free Software
#    Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA
#
use Shell qw(xm);
use Nagios::Plugin;
use Data::Dump qw(dump);

my $np = Nagios::Plugin->new(
    shortname => "XEN",
    usage     => "XEN status\n%s\n[-w],[-c],[-v]"
);

$np->add_arg(
    spec => "warning|w=s",
    help => "-w, --warning=range range<min,max>, Warning if current  instance number is outside of warning range\n" . " EXAMPLE\n" . " If -w 3:6 -c 2:9 and number of instances :2\n" . " raises warning alert as outside of the warning range and inside of critical rannge\n" . " any value lies in warning range and critical range should alert OK",
);

$np->add_arg(
    spec => "critical|c=s",
    help => "-c, --critical=range range<min,max>,critical if current running instances number is outside critical range\n" . " EXAMPLE\n" . " If -w 3:6 -c 2:9 and number of instances :1\n" . " raises critical alert as outside of critical range.",
);

$np->getopts;
my $warning  = $np->opts->warning;
my $critical = $np->opts->critical;

my $data = xm("list") or die $np->nagios_exit( UNKNOWN, "Can not execute xm command" );

#print dump $data;
#removes all special charachters and keeps those are listed in the bracket.
#remember there is space in the end after 0-9 which is not removed from $data
$data =~ s/[^a-zA-Z0-9 \n ]*//g;

#print dump $data;

my @array = split( " ", $data );

#print dump(@array);

my @active_domains;
my $running_count = 0;
my $i             = 10;
foreach $a (@array) {

    if ( $array[$i] eq "r" ) {
        $running_count++;
        my $tmp_dom = "$running_count) " . $array[ $i - 4 ] . ". ";
        push( @active_domains, $tmp_dom );
    }
    $i = $i + 6;
}

#print "runnig vm: ". $running_count;
my $code = $np->check_threshold(
    check    => $running_count,
    warning  => $np->opts->warning,
    critical => $np->opts->critical,

);

#adding performance data
$np->add_perfdata(
    label => 'active_domains',
    value => $running_count,

    # threshold => $code,
);
if ( $running_count == 0 ) {
    $np->nagios_exit( $code, "No domains are currently active" );
}

$np->nagios_exit( $code, "$running_count domains are currently active. active domains: @active_domains " );
1
```

#### check_xen_state
https://secure.opsera.com/wsvn/wsvn/opsview/trunk/opsview-core/nagios-plugins/check_xen_state

```
#!/usr/bin/perl
#
# AUTHORS: Rohit Deshmukh
#   Copyright (C) 2003-2010 Opsera Limited. All rights reserved
#
#    This file is part of Opsview
#
#    Opsview is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    Opsview is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with Opsview; if not, write to the Free Software
#    Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA
#

use Shell qw(xm);
use Nagios::Plugin;
use Data::Dump qw(dump);

my $np = Nagios::Plugin->new(
    shortname => "XEN STATUS",
    usage     => "to check XEN_STATUS\n%s\n[-n],[-w],[-o],[-v]\n",
    extra     => "
    Example:
            to alert CRITICAL if VM is in blocked state
            check_xen_state -n vmname -c blocked

            to alert WARNING if VM is in paused or blocked state
            check_xen_state -n vmname -w paused,blocked

            to define warning list and ok list together
            check_xen_state -n vmname -c dying -w paused,blocked

            Note:any vm state outside of the warning and critical list will generate ok alert.
                :if warning and critical list not defined then pligin will always alert ok.

            valid states running, blocked, paused, 'shut off', crashed, dying",
);

$np->add_arg(
    spec => "warning|w=s",
    help => "-w, --warning=list:list  Warning if current  instance state is in warning list ",
);

$np->add_arg(
    spec => "critical|c=s",
    help => "-c, --critical=list:list. critical if current  instance state is in critical list",
);
$np->add_arg(
    spec     => "name|n=s",
    help     => "-n, --name=STRING:STRING. index of the instacne",
    required => 1,
);

$np->getopts;

#valid states array
my %valid_states = (
    r => "running",
    b => "blocked",
    p => "paused",
    s => "shut off",
    c => "crashed",
    d => "dying"
);

# print dump(%valid_states);
my $warning  = $np->opts->warning;
my $critical = $np->opts->critical;

my $vm_name = $np->opts->name;
my $vm_data;
my $logfile = "check_xen_state.txt";

#vm_state=xm "list crummock";
#print dump $vm_state;

open STDERR, ">/tmp/$logfile"
    or die $np->nagios_exit( UNKNOWN, "Can not write to the /tmp/$logfile" );
$vm_data = xm "list $vm_name"
    or die $np->nagios_exit( UNKNOWN, "Can not execute xm command" );
close STDERR;

#removing white apaces from the out put
#$vm_state =~ s/^\s+//;
#$vm_state =~ s/\s+$//;
#print dump $vm_state;

$vm_data =~ s/[^a-zA-Z0-9\n ]*//g;

my @array_data = split( " ", $vm_data );

#print dump @array_data;

my $vm_id        = @array_data[7];
my $vm_mem       = @array_data[8];
my $vm_ini_state = @array_data[10];
my $vm_state     = $valid_states{$vm_ini_state};

#print dump $vm_state;

if ( $vm_state eq "" ) {
    $np->nagios_exit( UNKNOWN, "VM name $vm_name not found" );
}

my @warning = split( ",", $warning );

# print dump(@warning);

my @critical = split( ",", $critical );

# print dump(@ok);

#creating hash lists
my %warning_list = map { ( $_ => warning ) } @warning;

#print dump(%warning_list);

my %critical_list = map { ( $_ => ok ) } @critical;

# print dump(%critical_list);
#checking for valid states4
if ( exists( $warning_list{$vm_state} ) ) {
    $np->nagios_exit( WARNING, "VM name:-$vm_name current state is $vm_state, VM id:-$vm_id, VM memory:- $vm_mem " . "\n" );
}
if ( exists( $critical_list{$vm_state} ) ) {
    $np->nagios_exit( CRITICAL, "VM name:- $vm_name current state is $vm_state, VM id:-$vm_id, VM memory:- $vm_mem " . " \n" );
}
$np->nagios_exit( OK, "VM name:- $vm_name current state is $vm_state, VM id:-$vm_id, VM memory:- $vm_mem " . "\n" );
1
```
