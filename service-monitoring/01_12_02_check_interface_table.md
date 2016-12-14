# check_interface_table

## check_interface_table_v2.pl
* https://git.netways.org/plugins/check-interface-table
* https://www.netways.org/projects/plugins/files

```
# apt-get install libconfig-general-perl
```

> PERL V5 installed
>
> You need a working perl 5.x installation. Currently we use V5.8.8 under
> SUSE Linux Enterprise Server 10 SP1 for development. We know that it
> works with other versions, too.

### Test
Localhost, with snmpd configured and running.

```
$ perl check_interface_table_v2.pl -H localhost -C public --HTMLDir /home/michi/temp/
<a href="/nagios/interfacetable/localhost-Interfacetable.html">1 interface(s), 0 free</a>|check_interface_table::interface_table::time=1s;;;; uptime=316390s;;;; watched=0c;;;; useddelta=488s;;;; ports=1c;;;; freeports=0c;;;; AdminUpFree=0c;;;;
```

### Warnings
If you get deprecation warnings like this

```
Using an array as a reference is deprecated at check_interface_table_v2.pl line 2251.
Using an array as a reference is deprecated at check_interface_table_v2.pl line 2259.
```

you can silence that by editing the file and add in the third line

**Warning:** That can, or cannot work with newer versions. With Perl 5.14.2 it does.

```
#!/usr/bin/perl
# nagios: -epn
no warnings 'deprecated';
=head1 NAME
```

## Interfacetable_v3t

http://www.tontonitch.com/tiki/tiki-index.php?page=Nagios+plugins+-+interfacetable_v3t
