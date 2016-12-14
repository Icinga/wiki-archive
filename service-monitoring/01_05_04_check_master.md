# check_master

**Warning:** Sort of Failover Redunancy Script Collections. Incomplete.

> Add the Icinga slave script. This script (to run as cron task) checks for Icinga master status.
>
> If it's not OK twice, then the failover instance is enabled.
>
> On the other hand, while failover is running, if master goes back to life, failover is disabled again.
>
> This allows auto recovery of Icinga monitoring.

http://svn.reactos.org/svn/project-tools/trunk/nagios/check_master.sh

```
#!/bin/bash
#
# File:         check_master.sh
# Author:       Pierre Schweitzer <pierre@reactos.org>
# Created:      18 Jun 2012
# Licence:      GNU GPL v2 or any later version
# Purpose:      Script to check for master status. In case it is down
#		the slave is activated to take over.
#

MAIL=root
STATUS_FILE=/var/lib/icinga/rw/slave.status
CMD_FILE=/var/lib/icinga/rw/icinga.cmd
NRPE=/usr/lib/nagios/plugins/check_nrpe
MASTER=
CHECK=check_icinga

OLD_STATUS=`cat $STATUS_FILE`
# Status can be:
# 0: Not running
# 1: Running
# 2: About to run
# 3: About to stop

$NRPE -H $MASTER -c $CHECK 1>/dev/null 2>&1
STATUS=$?

if [ $STATUS -ge 2 ]; then
	if [ $OLD_STATUS -eq 2 ]; then
		NOW=`date +%s`
		# Reenable checks
		echo "[$NOW] START_EXECUTING_HOST_CHECKS;" >> $CMD_FILE
		echo "[$NOW] START_EXECUTING_SVC_CHECKS;" >> $CMD_FILE
		echo "[$NOW] ENABLE_NOTIFICATIONS;" >> $CMD_FILE

		echo "Slave taking over" | mail -s "Icinga: master down" $MAIL

		echo 1 > $STATUS_FILE
	elif [ $OLD_STATUS -eq 3 ]; then
		echo 1 > $STATUS_FILE
	elif [ $OLD_STATUS -eq 0 ]; then
		echo 2 > $STATUS_FILE
	fi
elif [ $STATUS -eq 0 ]; then
	if [ $OLD_STATUS -eq 3 ]; then
		NOW=`date +%s`
		# Disable checks
		echo "[$NOW] STOP_EXECUTING_HOST_CHECKS;" >> $CMD_FILE
		echo "[$NOW] STOP_EXECUTING_SVC_CHECKS;" >> $CMD_FILE
		echo "[$NOW] DISABLE_NOTIFICATIONS;" >> $CMD_FILE

		echo "Slave going back to sleep" | mail -s "Icinga: master up" $MAIL

		echo 0 > $STATUS_FILE

		# Reset Icinga state
		/etc/init.d/icinga reload
	elif [ $OLD_STATUS -eq 2 ]; then
		echo 0 > $STATUS_FILE
	elif [ $OLD_STATUS -eq 1 ]; then
		echo 3 > $STATUS_FILE
	fi
fi
```
