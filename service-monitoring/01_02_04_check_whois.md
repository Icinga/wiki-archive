# check_whois

**Warning:** This is just a basic overview what could be done with whois data

Grabbing the whois data can be useful to check if a domain is
* delegated to specific nameservers
* registrant / tech-c / admin-c are valid
* status is ok
* expiration date (if the whois output supports that)
* ...

There are some check_domain plugins out there, but they all require decisions on the tld being queried, so here we will only show some basics what check_tcp will help to fetch.
The whois protocol is a standard, therefore the initial handshake should work on each, querying port 43, going with correct line endings. Look up the RFC if you're interested.

To get an idea, how such data looks like, query the domain yourself.

```
# whois monitoring-portal.org
```

It will tell you the whois server to be queried (note that, we will use that in our check then).

```
[Querying whois.publicinterestregistry.net]
[whois.publicinterestregistry.net]
Access to .ORG WHOIS information is provided to assist persons in
determining the contents of a domain name registration record in the
Public Interest Registry registry database. The data in this record is provided by
Public Interest Registry for informational purposes only, and Public Interest Registry does not
guarantee its accuracy.  This service is intended only for query-based
access. You agree that you will use this data only for lawful purposes
and that, under no circumstances will you use this data to: (a) allow,
enable, or otherwise support the transmission by e-mail, telephone, or
facsimile of mass unsolicited, commercial advertising or solicitations
to entities other than the data recipient's own existing customers; or
(b) enable high volume, automated, electronic processes that send
queries or data to the systems of Registry Operator, a Registrar, or
Afilias except as reasonably necessary to register domain names or
modify existing registrations. All rights reserved. Public Interest Registry reserves
the right to modify these terms at any time. By submitting this query,
you agree to abide by this policy.
```

## check_tcp
The idea is simple - send a query string to the whois server, and expect and answer. If the answer matches an expected string, everything should be ok. Choose that string wisely!

If you are unhappy with the basic check_tcp output, write a little shell wrapper, parsing the output. This may especially help when parsing the expiration date - if provided.

check_tcp is part of the [Nagios Plugins](../plugins/01_02_nagios_plugins.md), so make sure to have those installed where you wanna run the check.

**Info:** Adding -v will enable verbose output, and you can debug the question / answer matching a lot easier!

### Config

#### Commands

```
define  command {
        command_name                    check_whois_monitoringportal
        command_line                    $USER1$/check_tcp -H whois.publicinterestregistry.net -t 30 -w 1.99 -c 7 -p 43 -E -s 'monitoring-portal.org\n' -e 'Status:OK' -j -d 2
}
```

#### Services
```
define service {
        name                            whois-active-template
# we use the icon_image hack
#        use                             pnp-svc
        check_period                    24x7
        check_command                   check_whois_monitoringportal
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
        use                             whois-active-template
        host_name                       HOSTWHOIS
        service_description             WHOIS monitoring-portal.org
        check_command                   check_whois_monitoringportal
}
```

#### Tests

```
# /usr/lib/nagios/plugins/check_tcp -H whois.publicinterestregistry.net -t 30 -w 1.99 -c 7 -p 43 -E -s 'monitoring-portal.org\n' -e 'Status:OK' -j -d 2
```
