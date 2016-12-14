# check_rrdcached

For the most recent version, get onto the Repository

https://github.com/formorer/nagios-plugins


```
#!/usr/bin/perl

# check_rrdcached - nagios plugin to check rrdcached via unixsocket
# Copyright (C) <2012>  Alexander Wirt <formorer@formorer.de>
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

use strict;
use warnings;
use Nagios::Plugin;

use IO::Socket;

my $np = Nagios::Plugin->new(
	usage => "Usage: %s [-t <timeout>] "
		."[-s <socketpath>]",
	version => '0.0',
	plugin  => 'check_rrdcached',
	timeout => 15,
);

$np->add_arg(
	spec => 'socketpath|s=s',
	help => '-s, --socketpath=FILENAME . Filename for the rrdached socket. Defaults to "/var/run/rrdcached.sock".'
);

$np->getopts;

#install an alarm to be able to timeout on a hanging socket
alarm $np->opts->timeout;

my $sock_path = $np->opts->socketpath || '/var/run/rrdcached.sock';

my $sock = IO::Socket::UNIX->new($sock_path);

$np->nagios_exit( CRITICAL, "CRIT: Could not connect to rrdached socket $sock_path: $!" ) unless $sock;

print $sock "STATS\n";
my ( $stats_expected, $stats_retrieved ) = 0;
my $response;
while (defined($_ = $sock->getline)) {
	if (/(\d+) Statistics follow/) {
		$stats_expected = $1;
	} else {
		if ( ! $stats_expected) {
			$np->nagios_exit( CRITICAL, "Illegal response from rrdcached: $_");
		}
		$response .= $_;
		$stats_retrieved++;
		last if $stats_expected == $stats_retrieved;
	}
}

$np->nagios_exit ( CRITICAL, "No response from rrdcached") unless $response;

#be nice to rrdcached
print $sock "QUIT\n";
close($sock);

foreach my $line (split("\n", $response)) {
	my ($key, $value) = split (/:\s*/, $line,2);
	$np->add_perfdata( label => $key, value => $value );
}

$np->nagios_exit( OK, "rrdcached up and alive");
```
