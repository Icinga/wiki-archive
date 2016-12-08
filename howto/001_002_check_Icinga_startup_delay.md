#Check Icinga Startup Delay

Between process start and actual event execution loop happens a lot. In case of having event broker modules loaded, they might start config dumps based on the initialization and reading.
To measure that value define a new command (based on check_dummy in order to prevent plugin dependency):

```
# 'check_icinga_startup_delay' command definition
define command {
       command_name    check_icinga_startup_delay
       command_line    $USER1$/check_dummy 0 "Icinga started with $$(($EVENTSTARTTIME$-$PROCESSSTARTTIME$)) seconds delay | delay=$$(($EVENTSTARTTIME$-$PROCESSSTARTTIME$))"
}
```

And then use it as service definition on localhost.

```
define service{
        use                             local-service         ; Name of service template to use
        host_name                       localhost
        service_description             Icinga Startup Delay
        check_command                   check_icinga_startup_delay
        notifications_enabled           0
        }
```

Perfdata is included, so you can actually graph the delay and test various thinks even better :-)
