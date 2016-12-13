# check_dnssec

There are various plugins out there, most of them require a dnssec validating resolver (like unbound provides).

**Warning:** This is by far incomplete. Help increase know how in this sector!


## dnssec_monitor
This is developed by the OpenDNSSEC group, in Ruby. It does not require a validating resolver, but within its libs. This can take long, especially when your NS record set is huge...

### Requirements
Ruby and Gems

```
# apt-get install ruby rubygems
```

and some special libs

```
# gem install text-format
# gem install dnsruby
```

### Source
I'm keeping a not very well synced in my github repo, so you can take that one as well. http://fisheye.opendnssec.org/browse/opendnssec/tags/monitor-0.0.5

```
$ cd ~/src ; git clone git://github.com/dnsmichi/dnssec_monitor.git
$ sudo cp -r dnssec_monitor /usr/lib/nagios/plugins
$ sudo chmod+x /usr/lib/nagios/plugins/dnssec_monitor/lib/nagios_dnssec.rb
```

### Config

#### Command

```
define  command {
        command_name                    check_dnssec
        command_line                    $USER1$/dnssec_monitor/lib/nagios_dnssec.rb -z $ARG1$ --kskcritical=$ARG2$ --kskwarn=$ARG3$ --zskcritical=$ARG4$ --zskwarn=$ARG5$
}
```

#### Service

```
define service {
        name                            dns-active-template
# we use the icon_image hack
#        use                             pnp-svc
        check_period                    24x7
        check_command                   check_dns
        contact_groups                  none
        notification_period             24x7
        check_interval                  5
        retry_interval                  1
        max_check_attempts              5
        is_volatile                     0
        parallelize_check               1
        active_checks_enabled           1
        passive_checks_enabled          1
        obsess_over_service             1
        event_handler_enabled           1
        flap_detection_enabled          1
        freshness_threshold             7200
        check_freshness                 1
        notification_options            u,w,c,r,f
        notifications_enabled           1
        notification_interval           20
        process_perf_data               1
        register                        0
        icon_image                      s.gif' width='0' height='0' border='0'></a><a href='/pnp4nagios/graph?host=$HOSTNAME$&srv=$SERVICEDESC$' class='tips' rel='/pnp4nagios/popup?host=$HOSTNAME$&srv=$SERVICEDESC$'><img width='20' height='20' src='/icinga/images/logos/Stats2.png' border='0
}

define  service {
        use                             dns-active-template
        host_name                       HOSTDNS
        service_description             DNSSEC dnssec-signiert.at
        check_command                   check_dnssec!dnssec-signiert.at!4!6!4!6
}
```

#### Tests

```
# /usr/lib/nagios/plugins/dnssec_monitor/lib/nagios_dnssec.rb -z dnssec-signiert.at --kskcritical=4 --kskwarn=6 --zskcritical=4 --zskwarn=6
DNSSEC OK:  : Finished checking on ns2.at43.at(78.104.145.251)
```
