# check_wmi_plus

**Warning:** needs review and howto

## General
http://www.edcint.co.nz/checkwmiplus/

## About

> ### [Overview](http://www.edcint.co.nz/checkwmiplus/?q=node/12)
>
> [Donations](http://www.edcint.co.nz/checkwmiplus/node/33) to this spare-time project appreciated.
>
> Check WMI Plus is a client-less Nagios plugin for checking Windows systems.
> No more need to install any software on any of your Windows machines. Monitor Microsoft Windows systems directly from your Nagios server.
>
> Check WMI Plus uses the Windows Management Interface (WMI) to check for common services (cpu, disk, sevices, eventlog...) on Windows machines. It requires the open source wmi client for Linux.
>
> Besides the built in checks, Check WMI Plus functionality can be easily extended through the use of an ini file. Check WMI Plus comes with a release version of the ini file, but you are free to add your own checks to your own ini file. Share your checks (you must be logged in) with others and get additional checksdeveloped by other users.
>
> In the future I plan to add more functionality along the lines of what NSClient++ can do - but with the advantage of being client-less. There will always be some things that NSClient++ can do and some things it can do better since it executes directly on the checked host, but check_wmi_plus is far simpler and has the major advantage of not requiring any software installations on any Windows machines.
>
> Monitor Windows Server 2000, 2003 and 2008, Windows XP, Windows 7, Windows Vista etc
>
> Can also monitor the following applications in great detail: Microsoft SQL Server, Microsoft SQL Express, Microsoft Exchange Server, Microsoft IIS, Server, Windows Terminal Services.
>
> Overall there are about 70 different checks (most with performance data) it can perform against your Windows servers.
>
> Here is a list of some of the checks it can do:
> 
> **Built-in checks:**
>
> checkcpu, checkcpuq, checkdrivesize, checkeventlog, checkfileage, checkfilesize, checkfoldersize, checkmem, checknetwork, checkprocess, checkservice, checkuptime, checkvolsize, checkwsusserver
>
> **Checks via the ini file configuration:**
>
> checkad, checkiis, checkts (terminal services), checkio, checkproc (additional process checks), checkeachcpu, checkdhcp, checkdns, checkprint, checkexchange, checksql, checksqlex (SQL Express), checkfolderfileage, checkusers, info
