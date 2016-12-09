#Setting up Icinga with IDOUtils on SuSE

These guides were written targetting **Icinga 1.x.**

##General

What will you get once Icinga with IDOUtils using this guide is installed?

* Icinga Core
* Icinga Classic UI (the "CGIs")
* Icinga IDOUtils
	* Mysql or Postgresql as database
* Icinga Docs

If you came here looking for **Icinga Web Setup**, please check: Setting up [Icinga Web on SuSE]().

##Preparations

Icinga is part of the default repository beginning with openSUSE 12.1 so no additional preparations are needed in this case.

To always get the latest Icinga packages or for SUSE LINUX Enterprise a separate repository is required. Check [Monitoring Repo](http://download.opensuse.org/repositories/server:/monitoring/) for the correct name of your distribution and then add the repository:
	
	# zypper ar http://download.opensuse.org/repositories/server:/monitoring/<distribution>


##Order of Installation

1. Install icinga
	
`# zypper in icinga`
	
2. Install the needed nagios-plugins

`# zypper in nagios-plugins-all`

3. If you need or want the classic gui, install icinga-www. If you want plain monitoring with icinga core and configure everything else by hand, you are done.

`# zypper in icinga-www`

3.1 An example user icingaadmin with password icingaadmin is installed to  `/etc/icinga/htpasswd.users`. To add a new basic auth user for apache run:

`# htpasswd /etc/icinga/htpasswd.users youradmin`


##Setting up IDOUtils

Optional you can install the Icinga IDOUtils. Icinga Data Output Utils are necessary for various database backed guis such as Icinga Web or Icinga Reporting. IDOUtils use the libdbi database abstraction layer and can use either MySQL or Postgresql by installing one of these packages:

```
# zypper in icinga-idoutils-mysql
# zypper in icinga-idoutils-pgsql
```

The steps to set up or upgrade the db are the same as in the guide Setting up Icinga with IDOUtils on RHELexcept that schema paths are as follow:

```
/usr/share/doc/packages/icinga-idoutils-mysql/mysql
/usr/share/doc/packages/icinga-idoutils-pgsql/pgsql
```

##Further Instructions

For additional information check the README.SUSE and README.SUSE.idoutils files which are shipped with the package and also available online:

[Icinga Readme](https://build.opensuse.org/package/view_file?file=README.SUSE&package=icinga&project=server%3Amonitoring)

[IDOUtils Readme](https://build.opensuse.org/package/view_file?file=README.SUSE.idoutils&package=icinga&project=server%3Amonitoring)