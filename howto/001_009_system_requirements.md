#System Requirements

**This page is under construction Please help edit or offer suggestions for deletion on the [support channels](http://www.icinga.org/support/).**

##Icinga server hardware minimum recommendations:
|No. monitored devices|No. CPUs|No. CPU Cores|CPU Speed|Memory|Hard disk capacity|Network|
|---|---|---|---|---|---|---|
|< 100|1|Dual-core|> 2GHz|4 GB|40 GB|100Mbit / 1Gbit|
|100 - 200|> 1|Dual-core|> 3GHz|4 GB|40 GB|100Mbit / 1Gbit|
|> 200|> 1|Dual-core|> 3GHz|> 4 GB|100 GB|100Mbit / 1Gbit|


**Note:** It is very hard to give reliable recommendations because several factors have to be taken into account:

- Important is not the absolute number of devices being monitored but the number of checks within a given time (e.g. five minutes). Let's imagine you have 1.200 devices being checked using a 30 minute interval. This way you have an average of 200 checks per 5 minutes. Given 100 devices being checked every minute you have an average of 500 checks per 5 minutes.
- Active checks require more resources (memory, CPU load) on your system than passive.
- Checks via ssh will consume additional resources.
- Additional addons (for graphing results or visualising information) might increase disk usage degrading overall system performance.
- 
System requirements increase with the number of monitored devices and associated services. The use of passive as opposed to active service checks can further improve system performance. See Icinga Performance Tuning for more.

Small environments might start with Icinga being installed in a virtual machine. Increasing latencies might indicate to move to a physical machine.

For UNIX based systems that are tied to a Windows operating system, VMWare is recommended for virtualisation.


##Notes on hardware needs by Icinga / Nagios users
As hardware requirements are unique to each environment, it is often good to draw on the experiences of other Icinga / Nagios users. We have found the following blog articles, and would welcome more - feel free to add to the blog list or table of hardware usage, or post in the comments:

[GERMAN Pimp my Icinga by Jens Schanz](http://blog.jensschanz.de/?p=1171)

[GERMAN Nagios Performance Test](http://www.pnp4nagios.org/nagios_performance_test) 

[GERMAN Serversupportforum.de - Chat forum discussion](http://serversupportforum.de/forum/monitoring/49994-anforderungen-von-icinga.html)

##Supported platforms 

|Platform|Icinga Server|Icinga Agent|
|---|---|---|
|RedHat/CentOS|Supported - [Download Package](https://www.icinga.org/download/packages/)|Supported|
|SUSE|Supported - [Download Package](https://www.icinga.org/download/packages/)|Supported|
|Debian|Supported - [Download Package](https://www.icinga.org/download/packages/)|Supported|
|Ubuntu|Supported - [Download Package](https://www.icinga.org/download/packages/)|Supported|
|Gentoo|Supported - [Download Package](https://www.icinga.org/download/packages/)|Supported|
|FreeBSD|Supported - [Download Package](https://www.icinga.org/download/packages/)|Supported|
|Windows|  |via NSClient++ (More on Icinga Windows Agent)|

An Icinga agent is not required to monitor external network services such as FTP, SSH, HTTP, DNS, LDAP, etc.

