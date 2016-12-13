#Icinga Classic UI Testing
##Test - but what exactly?

On the Classic UI there are different layers to test. The CGIs are embedded into the HTML presentation layer. Main tests can happen on the webserver (interpreting the HTML output) and/or the shell passing various parameters to the CGIs.

##Debug Hints

If you are debugging with gdb or common tools, make sure that you are running the compiled binaries, not the installed ones. make install normally strips the debug symbols.
A gdb cheatsheet can be found here: [GDB Cheatsheet](http://www.yolinux.com/TUTORIALS/GDB-Commands.html)

##Building the source
###Configure

`./configure` behavior with different flags:

* set cgi location
* set users
* set libs/includes


###Make

	make cgis
	
###Make Install

	make install-cgis install-html install-config

##Check CGIs

* Backup your current configuration and try to set several things.
* Change literals, strings in the config, does it resolve in the expected presentation, or do you encounter something strange (special characters, encoding)?
* Does e.g. setting the authorization in cgi.cfg to a test user really behave like desired?
* Check back on the docs and try to set suggested GET parameters and configurations - does it really work?
* Try to set extrem values (absolute minimum/maximum) and watch the resulting behavior.
* Try to access non-existant host/services on the different cgis and watch their behavior
* Generate a test configuration with e.g. http://search.cpan.org/dist/Monitoring-Generator-TestConfig/
* Play around with random host behaviour

###Errors

* Segfault on the CGIs - why this?
* Are there any noticable memory leaks
* gcc -Wall on Makefile.in if not already set
* gdb, strace, valgrind

###Javascript Errors


##Debugging

cgi.cfg does not provide any debug logging, but the webservers log can be of help for identifying the error in the first place.

* Run the compiled cgis on the shell to get an idea, which environment variables need to be exported. examples are:

```
export REMOTE_USER=icinga
export QUERY_STRING=host=test1
export REQUEST_METHOD=GET
```
* Run the cgis (from source) on shell and call installed ones via apache - differences?
* Run the cgis with an empty QUERY_STRING
* Run strace (or truss -f on Solaris) or gdb like this

###normal

	$ export REQUEST_METHOD=GET ; export QUERY_STRING=; /home/icinga/icinga-core/cgi/status.cgi

###debug with gdb
	$ export REQUEST_METHOD=GET ; export QUERY_STRING=; gdb /home/icinga/icinga-core/cgi/status.cgi
	gdb> run
	gdb> bt full
**Hint:** If passing more than one param pair to QUERY_STRING, make sure to put it within double quotes. e.g.

	~/coding/icinga/icinga-core/cgi$ export REMOTE_USER=icinga;
	export REQUEST_METHOD=GET; export
	QUERY_STRING="show_log_entries=&host=c1-mail21&service=PING&timeperiod=last7days&smon=6&sday=1&syear=2011&shour=0&smin=0&ssec=0&emon=6&eday=30&eyear=2011&ehour=24&emin=0&esec=0&rpttimeperiod=&assumeinitialstates=yes&assumestateretention=yes&assumestatesduringnotrunning=yes&includesoftstates=no&initialassumedservicestate=0&backtrack=4"