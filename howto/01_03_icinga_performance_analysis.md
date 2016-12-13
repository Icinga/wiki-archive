#Icinga performance analysis
##System
Get an idea about your hardware and software.

* Hardware specs
	* possible bottlenecks for I/O and CPU
* Distribution used
* Packages installed
* Virtualization or bare metal
* Mail for notifications

###Addons

* Event Broker Modules in Icinga
* Serialized (possible blocking) Addons

##Graph it

You need to know what's going on Monitoring the Icinga Host will give you an idea what to check and graph to analyse that over time.

###Common Bottlenecks
####I/O

* Many processes writing/reading on the same disk. 
* Possible stale locks on the filesystem.
* Too many open file descriptors.

####Load

* database on the same host
* Other CPU consuming processes
* Forks are expensive

###Core Stats
Use icingastats tool to generate Core statistics. Use check_icingastats to graph that over time.

Manual run

	/usr/local/icinga/bin/icingastats

You will get something like this

```
Icinga Stats 1.6.1
Copyright (c) 2009 Nagios Core Development Team and Community Contributors
Copyright (c) 1999-2009 Ethan Galstad
Last Modified: 12-02-2011
License: GPL

CURRENT STATUS DATA
------------------------------------------------------
Status File:                            /var/icinga/status.dat
Status File Age:                        0d 0h 0m 18s
Status File Version:                    1.6.1

Program Running Time:                   0d 0h 10m 9s
Icinga PID:                             5016
Used/High/Total Command Buffers:        0 / 0 / 4096

Total Services:                         2003
Services Checked:                       1981
Services Scheduled:                     1981
Services Actively Checked:              2003
Services Passively Checked:             0
Total Service State Change:             0.000 / 0.000 / 0.000 %
Active Service Latency:                 0.000 / 40.609 / 32.556 sec
Active Service Execution Time:          0.000 / 0.302 / 0.128 sec
Active Service State Change:            0.000 / 0.000 / 0.000 %
Active Services Last 1/5/15/60 min:     821 / 1968 / 1969 / 1969
Passive Service Latency:                0.000 / 0.000 / 0.000 sec
Passive Service State Change:           0.000 / 0.000 / 0.000 %
Passive Services Last 1/5/15/60 min:    0 / 0 / 0 / 0
Services Ok/Warn/Unk/Crit:              1943 / 40 / 0 / 20
Services Flapping:                      0
Services In Downtime:                   0

Total Hosts:                            209
Hosts Checked:                          206
Hosts Scheduled:                        206
Hosts Actively Checked:                 209
Host Passively Checked:                 0
Total Host State Change:                0.000 / 100.000 / 17.334 %
Active Host Latency:                    0.000 / 43.622 / 16.692 sec
Active Host Execution Time:             0.000 / 4.048 / 0.229 sec
Active Host State Change:               0.000 / 100.000 / 17.334 %
Active Hosts Last 1/5/15/60 min:        136 / 204 / 206 / 206
Passive Host Latency:                   0.000 / 0.000 / 0.000 sec
Passive Host State Change:              0.000 / 0.000 / 0.000 %
Passive Hosts Last 1/5/15/60 min:       0 / 0 / 0 / 0
Hosts Up/Down/Unreach:                  156 / 52 / 1
Hosts Flapping:                         100
Hosts In Downtime:                      0

Active Host Checks Last 1/5/15 min:     616 / 2604 / 2796
   Scheduled:                           77 / 471 / 511
   On-demand:                           539 / 2133 / 2285
   Parallel:                            320 / 1580 / 1709
   Serial:                              0 / 0 / 0
   Cached:                              296 / 1024 / 1087
Passive Host Checks Last 1/5/15 min:    0 / 0 / 0
Active Service Checks Last 1/5/15 min:  1289 / 6654 / 7081
   Scheduled:                           1289 / 6653 / 7080
   On-demand:                           0 / 1 / 1
   Cached:                              0 / 0 / 0
Passive Service Checks Last 1/5/15 min: 0 / 0 / 0

External Commands Last 1/5/15 min:      0 / 0 / 0
```

###Numbers

General Information on your monitoring systems (you should tell in case of asking about bad performance!)

* Total Hosts/Services
* Hosts/Services Checked
* Hosts/Services Scheduled
* Hosts/Services Actively Checked
* Hosts/Services Passively Checked

###Latency

Latency is bad. Execution Time is one the metrics indicating that there are long lasting checks causing the core to wait/hang/block. Think about passive checks with NSCA for that.

* Active Host/Service Latency 1/5/15 min
* Active Host/Service Execution Time 1/5/15 min
* Passive Host/Service Latency 1/5/15 min
* Passive Host/Service Execution Time 1/5/15 min

###Scheduler

How much is the scheduler able to schedule? 

* Active Host/Service Checks Last 1/5/15 min

###Passive
If passive commands are fed via external command pipe, this may result in 100% usage.

* Passive Host/Service Checks Last 1/5/15 min
* External Commands Last 1/5/15 min
