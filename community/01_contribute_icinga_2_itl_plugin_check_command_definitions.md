#Contribute Icinga 2 ITL Plugin Check Command Definitions

##Introduction

The Icinga Template Library (ITL) and its plugin check commands provide a variety of CheckCommand object definitions which can be included on-demand.

Advantages of sending them upstream:

* everyone can use and update/fix them
* one single place for configs and documentation
* developers may suggest updates and help with best practices
* you don't need to care about copying the command definitions to all 
* your instances (because the icinga2 package already provides them)


***While it may sound much the first time, you'll learn the more you contribute. If something is unclear, please ask in the community channels***

##Where do I start?

Get to know the check plugin and its options. Read the general documentation on how to integrate your check plugins and how to create a good CheckCommand definition.

[addon plugins](http://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/addons-plugins#plugins)

Or look into the existing configuration and "code"

[plugin check commands](http://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/plugin-check-commands#plugin-check-commands)

[ITL command definitions](https://github.com/Icinga/icinga2/blob/master/itl/command-plugins.conf)

A good command definition uses:

* command arguments including value, description, optional: set_if, required, etc.
* use comments **/* ... */** to describe difficult parts
* command name as prefix for the custom attributes referenced (e.g. "disk_")
* default values
	*  if host.address is involved, set a custom attribute (e.g. "ping_address") to the default "**$address$**". This allows users to override the host's address later on by setting the custom attribute inside the service apply definitions.

	* if the plugin is also capable to use ipv6, import "ipv4-or-ipv6" and use "**$check_address$**" instead of "**$address$**", this allows falling back to ipv6 if only this address is set.
if using set_if, set the variable used to false by default.
templates if there are multiple plugins doing the same (e.g. ping4 and ping6)

	* your love and enthusiasm in making it the perfect CheckCommand

##I have created a CheckCommand, what now?
Icinga 2 developers love documentation. This isn't just because we want to annoy anyone sending a patch, it's a matter of making your contribution visible to the community. Open the [Icinga Documentation](http://docs.icinga.org), search for the command definition, and already know about all its options.

So your patch should consist of 2 parts:

* the CheckCommand definition
* the documentation bits

Create a git clone of the master branch and work on that:

	$ git clone https://github.com/icinga/icinga2
	$ cd icinga2

##Add the CheckCommand Definition to Contrib Plugins
 
There already exists a defined structure for contributed plugins. Navigate to itl/plugins-contrib.d and check where your command definitions fits.
 
	$ cd itl/plugins-contrib.d/
	$ ls


If you're adding an existing Monitoring Plugin (some are still missing, see [#6225](https://dev.icinga.com/issues/6225)) please use itl/command-plugins.conf instead.
 
	$ cd itl/
	$ vim command-plugins-conf

###Existing Configuration File
Just edit it, and add your CheckCommand definition.

	$ vim operating-system.conf

Proceed to the documentation.

###New type for CheckCommand Definition

Create a new file with .conf suffix. 

	$ vim printer.conf

Add it into CMakeLists.txt as additional file in the FILES line (*alpha-numeric order*) making Icinga 2 install it.

	$ vim CMakeLists.txt
 
	-FILES ipmi.conf network-components.conf operating-system.conf virtualization.conf vmware.conf
	+FILES ipmi.conf network-components.conf operating-system.conf printer.conf virtualization.conf vmware.conf

Add the newly created file to your git commit.

	$ git add printer.conf

Do not commit it yet - first create the documentation.

##Create CheckCommand Documentation
Your contributed plugin should fit into one of the types inside the "plugins contrib" tree. Make sure to find the right spot you've chosen for the command definition before.

[plugins cotrib](http://docs.icinga.org/icinga2/latest/doc/module/icinga2/chapter/plugins-contrib#plugins-contrib)

Edit the according *.md inside the doc/ tree (numbers might change!):

	$ cd doc/
	$ vim 7-icinga-template-library.md
	

Get to know Markdown. Use [dillinger.io](http://dillinger.io) for online editing and read the FAQ about it: [Markdow FAQ](https://exchange.icinga.org/faq#markdown!)

Create a section for your plugin, add a description and a table of parameters. Each parameter should have at least:

* optional or required
* description of its purpose
* the default value, if any

Example for 'mem' Check added in 2.3:

## <a id="plugins-contrib-operating-system"></a> Operating System

In this category you can find plugins for gathering information about your operating system or the system beneath like memory usage.

### <a id="plugins-contrib-command-mem"></a> mem

The plugin `mem` is used for gathering information about memory usage on linux and unix hosts. It is able to count cache memory as free when comparing it to the thresholds. It is provided by `Justin Ellison` on [https://github.com](https://github.com/justintime/nagios-plugins). For more details see the developers blog [http://sysadminsjourney.com](http://sysadminsjourney.com/content/2009/06/04/new-and-improved-checkmempl-nagios-plugin).

Custom Attributes:

Name         | Description
-------------|-----------------------------------------------------------------------------------------------------------------------
mem_used     | **Optional.** Tell the plugin to check for used memory in opposite of **mem_free**. Must specify one of these as true.
mem_free     | **Optional.** Tell the plugin to check for free memory in opposite of **mem_used**. Must specify one of these as true.
mem_cache    | **Optional.** If set to true plugin will count cache as free memory. Defaults to false.
mem_warning  | **Required.** Specifiy the warning threshold as number interpreted as percent.
mem_critical | **Required.** Specifiy the critical threshold as number interpreted as percent.
 
##CheckCommand and Documentation added, what's next?

Create a new issue at [Icinga2 Bugtracker](https://dev.icinga.org/projects/i2/issues/new) if not already existing. Describe the feature request and note the issue number.
Commit your changes and create a git patch. The git commit message should contain a short subject, newline, description and issue reference.

	$ git status
	$ git commit -av
	Add printer CheckCommand definition
 
	refs #5647
 
	$ git format-patch -1

Attach the patch to the previously created issue and wait for developers to review & apply your patch. Thanks for your contribution! :-)
##Review and Patch Update Requirement
Don't be sad when the developers ask you to change your patch - you can just continue to work locally, and amend your changes later.
<additional changes>

	$ git commit -av --amend
	Add printer CheckCommand definition

	Updated from devs feedback

	refs #5647
 
	$ git format-patch -1
 