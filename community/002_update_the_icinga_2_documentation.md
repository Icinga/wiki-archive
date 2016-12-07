#Update the Icinga 2 documentation

Compared to Icinga 1.x and the docbook xml format the Icinga 2 documentation is fairly easy to edit - we use Markdown for that.

##Get the current documentation

The easiest way is to clone the current git master and edit all files inside the doc/ directory tree.

	$ git clone https://github.com/icinga/icinga2
	$ cd icinga2
	$ cd doc
 
You'll find the entire documentation split into numbered files for each chapter. Choose the one you want to edit.

##Edit a file

Open the file you want to edit in your favorite editor (e.g. vim, nano or any other graphical editor). Do your modifications.

###Markdown Hints

Since we also use Markdown inside the Icinga exchange plugins, please refer to this FAQ: [Markdown FAQ](https://exchange.icinga.org/faq#markdown)

[dillinger.io](http://dillinger.io) is an online markdown editor which also previews the written text into actual rendered markdown.

Alternatively you can include the Icinga 2 documentation into the Icinga Web 2 doc module. The Vagrant Boxes do that already at [Icinga Vagrant Boxes](https://github.com/icinga/icinga-vagrant)

##Create a Patch

Creating a git patch is fairly simple: Add the changes to your commit and add a telling commit mesage. Then save the git patch into a file.

	$ git add doc/2-getting-started.md
	$ git commit -v -m "Documentation: Update Getting Started"
	$ git format-patch -1
	
This will created a file called "0001-Documentation-....".

**Alternatively** you can create a backup before modifying the original issue and then create a diff patch:

	cp doc/2-getting-started.md doc/2-getting-started.md.orig
	<do your modifications and save>
	diff -ur doc/2-getting-started.md.orig doc/2-getting-started.md > update-	documentation.patch
 
##Create an issue and add your patch

Register an Icinga account if you haven't done so already: [Create an Icinga account](https://accounts.icinga.org/register)

Log into [Icing2 Bugtracker](https://dev.icinga.org/projects/i2) and create a new issue describing the original problem.

Attach the git patch file and upload it. That's it - now wait for the developers to say hi and apply your patch :-)

PS: Don't worry if the developers won't accept your patch but ask for modifications - quality comes first and you won't regret it later finding your modifications inside the official docs ;-)
