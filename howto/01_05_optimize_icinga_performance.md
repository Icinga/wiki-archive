#Optimize Icinga Performance


**This howto was written for Icinga 1.x.**

If you're looking for Icinga 2 performance tuning, please head over to [Icinga Docs](https://docs.icinga.org)

##[Analysis](01_03_icinga_performance_analysis.md) comes first

##Performance Hints

###Ramdisk
Make sure to have a ramdisk for checkresults, spool, temporary files such as performance data.
####Plugins
* use check_icmp instead of check_ping for
* Hosts: check-host-alive
* Services: Ping

####Core
* Check Result Reaper
	* set check_result_reaper_frequency=1
	* set max_check_result_reaper_time=30 (not too low, otherwise checks cannot get processed, and get orphaned)
* External Commands (Passive Checks Results)
	* run icingastats to get an idea how many passive checks are run in 5 minutes
	* verify that external command pipe is not stuffed
	* set external_command_buffer_slots= to an accurate value
	* set command_check_interval=-1 to reap as often as possible
####CPU
If you happen to have CPU cycles free for all, assign more to icinga

```
# renice -5 -p <pid>

# renice -5 -p `pidof icinga`
```
