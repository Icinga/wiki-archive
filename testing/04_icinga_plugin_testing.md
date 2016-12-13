#Icinga Plugin Testing
##General
This guide is intended to help you with running commands as the core would execute those.
 
Testing & Debugging your plugins should be done manually in the first place to reduce the possibilities of possible bugs.

Running a command can be one of the following:

* a **plugin** executed for a check
* a **notification** triggered
* an **eventhandler** triggered

Check below on how to extract the commandline and actually run it how the core would do it.

##Hints
**Never run plugins as root, but follow the instructions below!!!**

Icinga Core will be started as root, but after reading icinga.cfg and resource.cfg it will drop privilegues and following operations will be done as

* icinga_user
* icinga_group

as specified in icinga.cfg

Icinga Core can source various exported environment variables via init script includes.

**Make sure that the plugin follows the Plugin API**
##Raw commandline
See Icinga Core Debug Config and set the level to checks to see the raw command line output. 

**Starting with Icinga 1.6 Classic UI, you can allow users to translate to the raw commandline via command expander in "View Config"**

##Run the plugin
First, fetch the complete command line, either within the GUI (Classic UI provides a command expander within "View Config") or by hand, translating the macros yourself.

###Macros

If your command definition contains macros, make sure to translate them accordingly. (n stands for a number, depending on your configuration)

* Always fetch $USERn$ definition from your resource.cfg and translate that on the full plugin path
* Make sure to translate all $ARGn$ macros accordingly - see http://docs.icinga.org/latest/en/macrolist.html#macrolist-arg
* interpret host and service macros correctly
	* $HOST... oder $SERVICE... defines the object this macro is attached to
	* ...ADDRESS$ oder ...PROBLEMID$ defines the attribute in your host or service config you need to actually replace with its value
* Verify that this is the exact same source, as e.g. the debug log would tell you as well
resource.cfg

```
$USER1$=/usr/lib/nagios/plugins

yourcommandandhostdef.cfg

define command{
        command_name    check-host
        command_line    $USER1$/check_ping -H $HOSTADDRESS$ -w $ARG1$,$ARG2$ -c $ARG3$,$ARG4$ -p $ARG5$
        }

define host{
        name            bumsti
        address         1.2.3.4
        check_command   check-host!3000.0!80%!5000.0!100%!5
```        
translates into

	/usr/lib/nagios/plugins/check_ping -H 1.2.3.4 -w 3000.0,80% -c 5000.0,100% -p 5

###Execute on the shell


* change to the actual user
* clear the environment
* run the plugin

```
# su - icinga
$ env -i
$ /usr/local/icinga/libexec/check_ping -H foo.bar.com ....
```

if changing the user is denied (e.g. in debian by default), the main reason is security awareness of the packager/sysadmin.

* ask your sysadmin if you can temporarly assign a shell to the icinga user
	* change /bin/false or /sbin/nologin to /bin/sh or /bin/bash in /etc/passwd - **caution - don't screw that file up, better create a backup!!!**
	* save and retry
* after you are done testing, disable the shell for that user again!

Another method might be doing it as root, with `sudo -u user` command

```
# sudo -u icinga /usr/local/icinga/libexec/check_ping -H foo.bar.com
```

###Return Code
Each plugin exits with a state, which is interpreted by Icinga Core being a valid one for the defined states (otherwise you will most likely see the infamous "return code is out of bounds" which will require a manual run to identify the root cause).

Directly after having run the plugin, run this command in order to fetch the integer value returned on plugin exit.

	$ echo $?

##Capture Output
[icinga nagios debugging/](http://terminalinflection.com/icinga-nagios-debugging/) 
[capture plugin](http://www.waggy.at/nagios/capture_plugin.htm)

You may use that wrapper script in order to catch *STDOUT* and *STDERR* from the check run, in order to identify errors by more detail.
 
##Special environment
It may happen that the icinga user is required to run e.g. an Oracle check, invoking sqlplus and therefore needing various environment variables to be set.

Or, if you require sudo permissions for the commands invoked by the check plugin.
### Passwordless sudo

That may be required when invoking commands directly on the remote server, using mechanisms like NRPE or check_by_ssh.

Example from check_disk_btrfs which requires root permissions on the btrfs binary.

```
# apt-get install sudo
# visudo

icinga ALL=(ALL) NOPASSWD: /usr/sbin/btrfs filesystem df /*
```

If a daemon like NRPE executes the command and calls sudo, make sure that **requiretty** is *disabled* for that specific user.
	
	Defaults:icinga !requiretty

**When testing as icinga/nrpe user, you will normally have your tty set after login. This will mean that the behaviour of sudo is different, in regards of requiretty detection e.g. - like a problem described [here](http://www.monitoring-portal.org/wbb/index.php?page=Thread&postID=99504&highlight=sudoers+tty+require#post99504).**

### Environment variables
So instead of doing env -i in the above example, make sure that the actually needed exported variables are all set.

**Don't export such things in .bashrc or similar - include it in the init script for proper export as the core daemon won't actually login to a shell, only call popen/execvp**

The default initscript uses

	/etc/sysconfig/icinga

to source the proper exports of environment variables.

Example with ORACLE_HOME in PATH

	export ORACLE_HOME=/usr/lib/oracle/11.2/client64
	export PATH=$PATH:$ORACLE_HOME

### Environment Macros
It's not advised to use them, as they cause some more CPU Load as well as possible memory leaks on the system, but in order to debug or testdrive them, you can dump the icinga user's environment on command execution.

An example how to do so is available in [GIT for Issue #3322](https://git.icinga.org/?p=icinga-core.git;a=blob;f=tests/etc/3322.cfg;hb=HEAD). Instead of your buggy command_line you could put this as a replacement, or executed just before the notification or check command execution.

```
define command {
        command_name    		3322check_env
        command_line    		/usr/bin/env >> /tmp/3322_icinga.$TIMET$.env
}
```
### Manual environment variables export

Exporting environment variables manually is pretty easy, But do not confuse them with the core's macros. They are computed within the core and cannot be simulated manually!

	$ . /etc/sysconfig/icinga

and verify it by e.g.

	$ echo $ORACLE_HOME
	$ echo $PATH
	
After having done that, run the plugin as icinga user.

	$ /usr/local/icinga/libexec/check_oracle_health ....

