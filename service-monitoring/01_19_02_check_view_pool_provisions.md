# check_view_pool_provisions

## Requirements
* check_nrpe PlugIn on Icinga Monitoring Server
* NSCclient++ or NSCP on a VMware View Connection Server
* VMware View PowerCLI on a VMWare View Connection Server
* Windows Powershell on the VMWare View Connection server

## Usage
The plugin needs to run on one of the VMware View Connection Servers. It checks the number of provisioned Desktops of a specific view pool.

You can specify how many Desktops need to be available until the warning and critical alerts are triggered.

The check plugin is triggered via check_nrpe towards nsclient++ aka. nscp.  So NRPE Server Support must be enabled.

The PlugIn delivers Icinga performance data in a PNP (http://pnp4nagios.org) compatible format

```
NRPEServer = 1
```

Furthermore you need to allow arguments for nrpe and check_external script module (nscp config style - you need to change it for older nsclient (< 0.4.0) versions)

`nsclient.ini`
```
\[/settings/NRPE/server\]
allow arguments = 1
performance data = 1

\[/settings/external scripts\]
allow arguments = 1

\[/settings/external scripts/scripts\]
check_view_pool_provisions=cmd /c echo scripts/check_view_pool_provisions.ps1 $ARG1$ $ARG2$ $ARG3$; exit $lastexitcode \| powershell \-NoLogo \-InputFormat none \-NoProfile \-NonInteractive \-Command -
```

The check command on your Icinga box will be:

```
./check_nrpe \-H view.connection.server \-t 30 \-c check_view_pool_provisions \-a pool_id warn crit
```

## Defaults
* warning = Pool maximum - 2
* critical = Pool maximium

## Example
In this case a warning is triggered if you 5 desktops available in your pool and critical is triggered if there are only 2 left.

```
./check_nrpe \-H view.connection.server \-t 30 \-c check_view_pool_provisions \-a pool_id 5 2
```

The result could be:
* `OK: 1 of 5 available|'desktops'=1`
* `WARNING: 1 of 5 available|'desktops'=1`
* `CRITICAL: 1 of 5 available|'desktops'=1`
