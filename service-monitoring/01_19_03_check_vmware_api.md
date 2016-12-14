# check_vmware_api

## Plugin
http://exchange.nagios.org/directory/Plugins/Operating-Systems/*-Virtual-Environments/VMWare/check_vmware_api/details

## Hints
**SSL error on certificate verification**

Set the following on your script

```
$ENV{PERL_LWP_SSL_VERIFY_HOSTNAME} = 0;
```

If that does not help, Version 5.x CLI may require another method described here:

* http://communities.vmware.com/message/2225675 - use an older LWP version which did not deprecate/ignore  PERL_LWP_SSL_VERIFY_HOSTNAME
* http://blog.jensschanz.de/?p=1187 (IO::Socket::SSL and newer LWP versions)

> Nach der Installation klappt der Verbindungaufbau zum VMWare-Server leider nicht, da das installierte LWP::Protocol::https das “fehlerhafte” Zertifikat nicht mag und mit folgendem Fehler terminiert:
>
> ```
> UNKNOWN: ERRORMSG: Server version unavailable at 'https://localhost:443/sdk/vimService.wsdl' at /usr/share/perl/5.10/VMware/VICommon.pm line 545.
> ```
>
> Damit das doch funktioniert, muss in dem Perl-Modul SSL_verify_mode von 1 nach 0 umgestellt werden. Damit werden dann auch ungültige Zertifikate akzeptiert.
>
> `/usr/local/share/perl/5.10.1/LWP/Protocol/https.pm`
>
> ```
> my %ssl_opts = %{$self->{ua}{ssl_opts} || {}};
> if (delete $ssl_opts{verify_hostname}) {
    > $ssl_opts{SSL_verify_mode} ||= 0;
    > $ssl_opts{SSL_verifycn_scheme} = 'www';
> }
> ```
