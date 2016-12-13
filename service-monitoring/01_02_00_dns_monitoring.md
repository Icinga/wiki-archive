# DNS Monitoring

## DNS

[RIPE Atlas Monitoring](01_02_03_ripe_atlas_monitoring.md)

## Checking all nameservers for zone are in sync
https://github.com/dwt/monitoring_probes/blob/master/check_nameservers_are_in_sync_for_zone.py

## DNSSEC

### NagVal
* https://github.com/jpmens/nagval

### dnssec monitor
* By the Opendnssec Group: [check_dnssec](01_02_02_check_dnssec.md)
* By .se registry: https://github.com/dotse/dnssec-monitor

### Zone RRSIG Expiration
* http://dns.measurement-factory.com/tools/nagios-plugins/check_zone_rrsig_expiration.html

### Zone Validation
* https://blog.kumina.nl/2013/08/on-our-new-dns-infrastructure-and-dnssec/ (requires libunbound and the python bindings)
* https://github.com/kumina/nagios-plugins-kumina/blob/master/check_dnssec_validation.py
* https://github.com/kumina/nagios-plugins-kumina/blob/master/check_dnssec_validation.py

## Whois
[check_whois](01_02_04_check_whois.md)
