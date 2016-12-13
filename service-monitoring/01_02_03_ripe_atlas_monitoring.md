# RIPE Atlas Monitoring

## General

https://atlas.ripe.net/about/

> RIPE Atlas is a global network of probes that measure Internet connectivity and reachability, providing an unprecedented understanding of the state of the Internet in real time.
There are currently several thousand active probes in the RIPE Atlas network, concentrated in the RIPE NCC service region of Europe, the Middle East and parts of Central Asia, and the network is constantly growing. The RIPE NCC collects the data from this network and provides useful maps and graphs based on the aggregated results. RIPE Atlas users who host a probe can also use the entire RIPE Atlas network to conduct customised measurements that provide valuable data about their own network(s).

https://labs.ripe.net/Members/suzanne_taylor_muzzin/introducing-ripe-atlas-status-checks

## Konfiguration

### Icinga 1.x

https://raw2.github.com/RIPE-Atlas-Community/ripe-atlas-community-contrib/master/scripts_for_nagios_icinga_alerts

```
define service {
     use generic-service
     host_name Ripe_atlas
     service_description Test_Atlas
     check_command check_http!-I atlas.ripe.net  -r 'global_alert":false' --ssl=1 -u /api/v1/status-checks/1040425/?permitted_total_alerts=1
}
```

### Icinga 2.x
Shown on the blog.

```
object CheckCommand "ripe-atlas" {
  import "plugin-check-command"
  command = [ PluginDir + "/check_http",
              "-I", "$apiaddress$",
              "-r", "$apiresult$",
              "--ssl=1",
              "-u", "/api/v$apiversion$/status-checks/$measureid$/?permitted_total_alerts=$critalerts$",
              "-t", "$timeout$"
  ]
  vars.apiaddress = "atlas.ripe.net"
  vars.apiresult = "global_alert\":false"
  vars.apiversion = "1"
  vars.critalerts = "1"
  vars.timeout = 30
}

object Host "RIPE Atlas" {
  import "generic-host"
  address = "atlas.ripe.net",
  address6 = "2001:67c:2e8:22::c100:69e"
  vars.measureid = "1040425"
}

object Host "RIPE Lab" {
  import "generic-host"
  address = "labs.ripe.net",
  address6 = "2001:67c:2e8:22::c100:699"
  //uncomment to apply service
  //vars.measureid = "1040425"
}

apply Service "atlas" {
  import "generic-service"
  check_command = "ripe-atlas"
  vars.apiresult = "global_alert\":false"
  vars.critalerts = "1"
  vars.measureid = "$host.vars.measureid$"
  assign where match("RIPE*", host.name)
  ignore where !host.vars.measureid
}
```
