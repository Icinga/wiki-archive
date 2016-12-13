# Eventhandlers

## Documentation
http://docs.icinga.org/latest/en/eventhandlers.html

## Examples
Icinga provides various eventhandlers in the source tarball, as well as packages may install that as examples too.

**Examples from the community**:

Test flow of mail based on an external echo service through EWS (thanks to @sperrgebiet42): http://gallery.technet.microsoft.com/Test-the-Mailflow-based-on-52edd45b

## Tarball

```
contrib/eventhandlers/
```

**Warning:** Make sure to set **--with-ext-cmd-file-dir=<path>** correctly, as that path will be set in the eventhandlers too.

```
$ ./configure --with-eventhandler-dir=<path>
$ make install-eventhandlers
```

## Debian

```
/usr/share/icinga/plugins/eventhandlers/
```

## RHEL

```
/usr/lib{64}/icinga/eventhandlers
```

## SuSE

```
/lib/icinga/eventhandler
```
