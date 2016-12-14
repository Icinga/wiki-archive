# check_view_pool

## Purpose
This Plugin checks if a VMware View Desktop Pool has been disabled due to provisioning errors. It needs to run on one of the VMware View Connection Servers.

## Requirements
This Plugin uses the VMware View PowerCLI. The PowerCLI must be available on the View Connection Server the Plugin is run on.

The check plugin is triggered via check_nrpe towards nsclient++ aka. nscp so nscp or the older nsclient++ must be installed and running.
Therefor you need to enable NRPE Support in nsclient.ini

NRPEServer = 1 (is the directive for nscp)

Furthermore you need to allow arguments for nrpe and check_external script module (nscp config style - you need to change it for older nsclient (< 0.4.0) versions)

```
[/settings/NRPE/server]
allow arguments = 1
performance data = 1

[/settings/external scripts]
allow arguments = 1
```

## sage

```
[/settings/external scripts/scripts]
check_view_pool=cmd /c echo scripts/check_view_pool.ps1 $ARG1$; exit $lastexitcode | powershell -NoLogo -InputFormat none -NoProfile -NonInteractive -Command -
```

The check command on your Icinga box will be:
`./check_nrpe -H <host> -t 30 -c check_view_pool -a pool_id`

The result will be either:
* `OK: Pool pool_id is enabled. |'enabled'=1`
* `CRITICAL: Pool pool_id is disabled or doesn't exist.|'enabled'=0`

Have a lot of fun!
