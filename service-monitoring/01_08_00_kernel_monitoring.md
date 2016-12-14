# Kernel Monitoring

## Check Running Kernel
Compare loaded with installed kernel - in order to detect possible reboots missing after dist-upgrades.

## Requirements
* NRPE
* http://svn.noreply.org/svn/weaselutils/trunk/nagios-check-running-kernel

## Script

```
# vim /usr/lib/nagios/plugins/check_running_kernel

#!/bin/bash

# Check if the running kernel has the same version string as the on-disk
# kernel image.

# Copyright 2008 Peter Palfrader
#
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files (the
# "Software"), to deal in the Software without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject to
# the following conditions:
#
# The above copyright notice and this permission notice shall be
# included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
# EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
# NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
# LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
# OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
# WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

OK=0;
WARNING=1;
CRITICAL=2;
UNKNOWN=3;

get_offset() {
	local file needle

	file="$1"
	needle="$2"
	perl -e '
		undef $/;
		$i = index(<>, "'"$needle"'");
		if ($i < 0) {
			exit 1;
		};
		print $i,"\n"' < "$file"
}

get_image() {
	local image GZHDR1 GZHDR2 off

	image="$1"

	GZHDR1="\x1f\x8b\x08\x00"
	GZHDR2="\x1f\x8b\x08\x08"

	off=`get_offset "$image" $GZHDR1`
	[ "$?" != "0" ] && off="-1"
	if [ "$off" -eq "-1" ]; then
		off=`get_offset "$image" $GZHDR2`
		[ "$?" != "0" ] && off="-1"
	fi
	if [ "$off" -eq "0" ]; then
		zcat < "$image"
		return
	elif [ "$off" -ne "-1" ]; then
		(dd ibs="$off" skip=1 count=0 && dd bs=512k) < "$image"  2>/dev/null | zcat 2>/dev/null
		return
	fi

	echo "ERROR: Unable to extract kernel image." 2>&1
	exit 1
}

searched=""
for on_disk in \
	"/boot/vmlinuz-`uname -r`"\
	"/boot/vmlinux-`uname -r`"; do

	if [ -e "$on_disk" ]; then
		on_disk_version="`get_image "$on_disk" | strings | grep 'Linux version' | head -n1`"
		[ -z "$on_disk_version" ] || break
		on_disk_version="`cat "$on_disk" | strings | grep 'Linux version' | head -n1`"
		[ -z "$on_disk_version" ] || break

		echo "UNKNOWN: Failed to get a version string from image $on_disk"
		exit $UNKNOWN
	fi
	searched="$searched $on_disk"
done

if ! [ -e "$on_disk" ]; then
	echo "WARNING: Did not find a kernel image (checked$searched) - I have no idea which kernel I am running"
	exit $WARNING
fi


running_version="`cat /proc/version`"
if [ -z "$running_version" ] ; then
	echo "UNKNOWN: Failed to get a version string from running system"
	exit $UNKNOWN
fi

if [ "$running_version" != "$on_disk_version" ]; then
	echo "WARNING: Running kernel does not match on-disk kernel image: [$running_version != $on_disk_version]"
	exit $WARNING
else
	echo "OK: Running kernel matches on disk image: [$running_version]"
	exit $OK
fi
```

## NRPE Config

```
# vim /etc/nagios/nrpe.d/icinga.cfg

# kernel
command[check_running_kernel]=/usr/lib/nagios/plugins/check_running_kernel

# service xinetd restart
```

## Icinga Check

```
# kernel
  ( ( "check_nrpe_command!check_running_kernel", "Kernel running", True ) , [ "cluster" ], ALL_HOSTS ),
```

## Tests

```
root@foo:/usr/lib/nagios/plugins# ./check_running_kernel
OK: Running kernel matches on disk image: [Linux version 3.2.0-0.bpo.2-amd64 (Debian 3.2.18-1~bpo60+1) (debian-kernel@lists.debian.org) (gcc version 4.4.5 (Debian 4.4.5-8) ) #1 SMP Sun Jun 3 21:40:57 UTC 2012]
```
