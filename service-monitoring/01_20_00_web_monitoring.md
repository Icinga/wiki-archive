# Web Monitoring

## Classic
check_http plugin provided with [Nagios Plugins](../plugins/01_02_nagios_plugins.md).

## Webinject
The project's maintainership was taken over by Sven Nierlein: https://github.com/sni/Webinject

http://labs.consol.de/nagios/check_webinject/

> check_webinject is a Nagios check plugin based on the Webinject Perl Module available on CPAN which is now part of the Webinject project. We use it heavily at ConSol and did a complete rework including some bugfixes and enhancements for Nagios 3.


### Archive
This remains outdated, but is the original plugin, for those seeking that one.

http://webinject.org/plugin.html

> __Why Another HTTP Plugin for Nagios?__
>
> There is an existing plugin for Nagios named 'check_http' that is installed along with the official set of Nagios plugins. The check_http plugin tests the HTTP service on a specified host and port. It has a variety of options and is useful for cursory monitoring of an HTTP service.
>
> However there are instances when it would be useful to do more than send a single request to test your HTTP service. WebInject is a test harness that can operate as an intelligent test agent that is able to chain together test cases to form a functional test suite for your web application or web service.
>
> For example, lets say you wanted to monitor a web application's functionality through multiple phases:
>
> * Phase 1 - Connect to the application
> * Phase 2 - Authenticate a user under the web application's login/authentication system
> * Phase 3 - Verify that you can navigate through the application while you're authenticated
> * Phase 4 - Do a sample request that accesses a database to verify it is available to the web application
>
> This could not be done in Nagios with a simple check_http which just verifies results of a single request without chaining tests that depend on previous responses. However, we can use WebInject in the Nagios plugin mode to do the job. The idea is to consider all of these 4 consecutive WebInject tests as one global test in the Nagios environment.
