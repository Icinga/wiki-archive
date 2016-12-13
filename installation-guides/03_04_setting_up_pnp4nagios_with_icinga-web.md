# Setting up PNP4Nagios with Icinga-Web

## Install pnp4nagios

First, checkout Setting up [PNP with Icinga](03_00_setting_up_pnp_with_icinga.md)

Install pnp4nagios as described in the pnp4nagios documentation.

Change pnp4nagios config to match your icinga config. You propably want to change these:

```
npcd.cfg
* user = icinga
* group = icinga
* log_file = /var/log/icinga/npcd.log
* perfdata_spool_dir = /var/icinga/spool/
* perfdata_file = /var/icinga/perfdata.dump

process_perfdata.cfg
* LOG_FILE = /var/log/icinga/perfdata.log

config.php
* $conf['nagios_base'|'nagios_base'] = "/icinga/cgi-bin";
```

as well as make sure that the apache configuration points to the correct htpasswd.users file

```
# vim /etc/apache2/conf.d/pnp4nagios.conf
        AuthUserFile /usr/local/icinga/etc/htpasswd.users

# service apache2 restart
```

**Warning:** Since the **PNP Graphs are embedded as iframes**, Icinga Web cannot easily pass its credentials to PNP. __Currently, there isn't an easy solution for that, other than updating the http auth__
A more difficult way in resolving this will be to __use an SSO provider like Kerberos/Shibboleth/etc and protect both, /icinga-web and /pnp4nagios with the same session configuration.__

## Templates Extension

**Preferred** This is the simplest way to do this. Valid from version 1.5 and above.

To leave grid templates untouched we've integrated XML extension to customize grid templates with simple snippets. PNP integration is made with these extension to be upgrade safe.

Get onto your Icinga Web source directory, copy the templates, and clear the cache.

```
$ sudo cp contrib/PNP_Integration/templateExtensions/pnp-* /usr/local/icinga-web/app/modules/Cronks/data/xml/extensions/
$ sudo /usr/local/icinga-web/bin/clearcache.sh
```

Icinga Web will then decide upon process_perf_data=1 being set on your hosts/services config where to show the graph icon.

### Packages
* On **Debian**, you'll need to install pnp4nagios as well as **icinga-web-pnp** being the template extensions (Backports recommended)
* On **RHE**, you'll need to to install pnp4nagios as well as **icinga-web-module-pnp** being the template extensions (Repoforge only currently)
* On **SuSE**, you'll need pnp4nagios, as well as **icinga-web-pnp_integration** as template extensions

## README
Excerpt from the INSTALL in contrib/PNP_Integration/INSTALL

```
cat contrib/PNP_Integration/INSTALL
************************
* INSTALLATION
************************

To install this addon, simply copy both xml files under templateExtensions to
your icinga-webs app/modules/Cronks/data/xml/extensions folder and clear the app/cache/CronkTemplates folder

To remove it, just delete the extension files and clear the cache folder again
```
