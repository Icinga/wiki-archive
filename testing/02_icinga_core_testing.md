#Icinga Core Testing
##Test - but what exactly?

On the core there are different layers to test. This splits up in several modules like IDOUtils but als in the installation steps and starting Icinga Core afterwards.
[Icinga IDOUtils Testing]() is a seperate section though.

The primary targets are:

* Configuration Verification
* Running the daemon
* Debugging

##Check Configuration

* Backup your current configuration and try to set several things.
* Change literals, strings, does it resolve in the expected behavior, or do you encounter something strange?
* Does e.g. setting the retry_interval to a test value really result like desired?
* Check back on the docs and try to set suggested values and configurations - does it really work?
* Try to set extrem values (absolute minimum/maximum) and watch the resulting behavior.
* Generate a test configuration with e.g. http://search.cpan.org/dist/Monitoring-Generator-TestConfig/
* Play around with random host behaviour



**Make sure to check objects.cache where the core stores the final compiled objects, instead of getting busy with templates!**

##Debug Hints

If you are debugging with gdb or common tools, make sure that you are running the compiled binaries, not the installed ones. make install normally strips the debug symbols.
A gdb cheatsheet can be found here: [GDB Cheatsheet](http://www.yolinux.com/TUTORIALS/GDB-Commands.html)

##Building the source
###Configure
`./configure` behavior with different flags:

* enable modules
* set prefix
* set users
* set libs/includes


###Make

Compiling different things, just type 'make' to see all available options

###Make Install

Test if different installing options do what they are expected to do e.g. fullinstall should do everything.

##Debugging

icinga.cfg provides debug settings. Set them to those values to get everything:

* debug_level=-1 (everything)
* debug_verbosity=2 (very detailed)
* debug_file=/where/you/have/enough/storage/icinga.debug
* max_debug_file_size=1000000000 (1GB)


##Running the Core

* Are Macros processed correctly?
* How do the illegal chars in macros affect the string escaping?
* Host States are expected - e.g. host is down when you disable the network route temporarly?
* Service checks behave correctly
* External Commands are being processed
* Passive checks are working?
* Are the permissions on the command pipe set correctly, does it work like expected?
* Is the daemon running, but a dead daemon not processing any checks and results - what might have caused that, why does it hang?
* States are going from soft to hard, state changes are happening correctly?
* Notifications are being sent correctly
* Escalations are being handled they way they are expected, Escalation conditions defined work as they should

###Errors

* Segfault on the Core - why this?
* Faulty modules being loaded?
* Are there any noticable memory leaks
* gcc -Wall on Makefile.in if not already set
* gdb, strace, valgrind

##t/ and t-tap/
```
$ sudo yum install perl-HTML-Lint
$ sudo yum install perl-Test-WWW-Mechanize
```
```
$ ./configure
$ make test
```


##Plugins

* Matches the check config for the plugins, is there anything different from the docs to the real config for check commands?

##Addons
###NSCA

* NSCA daemon is processing passive check results
* NSCA client sends a check result, does that go through onto the core and Webinterface (cgi)

###NRPE

* Do Checkresults come back?
* Is there anything interferring with the communication to the core

###IDOUtils
Please check [Icinga IDOUtils Testing]()
##Classic UI / CGIs
Please checl [Icinga Classic UI Testing]()
##Found something?
Then please let us know: [Create an Issue]()

##What else?
Next to Icinga Core there are projects in Icinga, which needs to be tested altogether then.
Have a look into:

[Icinga IDOUtils Testing]()

[Icinga Web Testing]()

[Icinga Doc Testing]()