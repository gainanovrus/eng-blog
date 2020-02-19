---
layout: single
title:  Install and configure NTP on FreeBSD 11
categories: sysad
tags: ntp freebsd
header:
  teaser: /assets/images/ntp-freebsd-setup.png
  og_image: /assets/images/ntp-freebsd-setup.png
---

In this article will be described what use NTP to synchronize system time on FreeBSD.
I use FreeBSD 11 that ran as VPS in [Hetzner][hetzner] cloud.

Previously you should check what version is installed on your machine. Use for this command `ntpd --version`. Result:
```
ntpd 4.2.8p10-a (1)
```

If you have NTP server that version is less 4.2.7 you should update it because you might be [attacked][ntp-attack].

Using a [portsnap][portsnap] to update your local ports:
```
portsnap fetch extract
portsnap fetch update
```

After that instal `zoneinfo` package and copy selected local zone to change your timezone (my zone is `Europe/Moscow`:
```
cd /usr/ports/misc/zoneinfo && make install clean
cp /usr/share/zoneinfo/Europe/Moscow /etc/localtime
```

Editing your NTP configuration file `/etc/ntp.conf`:
```
restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery

restrict 127.0.0.1
restrict -6 ::1

server ntp1.hetzner.de iburst
server ntp2.hetzner.de iburst
server ntp3.hetzner.de iburst
```

Don't forget setup your servers in these parameters:
* `server ntp#.hetzner.de iburst` -- use this server to synchronize time; `iburst` keyword allows to speed up first connection

And now only need to add a command that will allow running NTP on startup ([rc.conf][rc.conf]):
```
echo ntpd_enable=\"YES\" >> /etc/rc.conf
echo ntpd_sync_on_start=\"YES\" >> /etc/rc.conf
```

Parameter `ntpd_sync_on_start` is setting `YES` to syncs the system's clock	on startup and to remove a restriction a lot time offset.

Finally lets start a local NTP server:
```
/etc/rc.d/ntpd start
```

Some moments ago your local system time will update and you can check a status on this:
```
# ntpq -p
remote refid st t when poll reach delay offset jitter
==============================================================================
+ntp1.hetzner.de 192.53.103.108 2 u 1 64 1 2.864 -0.889 0.153
+ntp2.hetzner.de 192.53.103.108 2 u 4 64 1 0.328 0.219 0.114
*ntp3.hetzner.de 192.53.103.108 2 u 2 64 1 0.306 0.832 0.092

# ntpdate -q localhost
server 127.0.0.1, stratum 3, offset -0.000008, delay 0.02568
server ::1, stratum 3, offset 0.000005, delay 0.02571
ntpdate[52224]: adjust time server 127.0.0.1 offset -0.000008 sec
```

If you want to sync onetime use this command:
```
# ntpdate -v -b ntp1.hetzner.de
ntpdate[11356]: ntpdate 4.2.8p10-a (1)
ntpdate[11356]: step time server 213.239.239.164 offset 0.004194 sec
```

## Additional information:
This links might useful to get any additional information:
* [Hetzner - Install the NTP daemon](https://wiki.hetzner.de/index.php/Uhrzeit_synchronisieren_mit_NTP/en)
* [ntpd -- NTP daemon	program](https://www.freebsd.org/cgi/man.cgi?query=ntpd&sektion=8&apropos=0&manpath=FreeBSD+11.1-RELEASE+and+Ports)
* [ ntp.conf -- Network Time Protocol (NTP) daemon configuration file format](https://www.freebsd.org/cgi/man.cgi?query=ntp.conf&sektion=5&apropos=0&manpath=FreeBSD+11.0-RELEASE+and+Ports)
* [Синхронизация часов через NTP](https://www.freebsd.org/doc/ru/books/handbook/network-ntp.html)
* [Настройка ntpdate/ntpd на FreeBSD 10](https://sysadmin-note.ru/nastrojka-ntpdatentpd-na-freebsd-10/)
* [Атака с помощью вашего сервера времени: NTP amplification attack (CVE-2013-5211)](https://habrahabr.ru/post/209438/)

[hetzner]: https://www.hetzner.de/
[ntp-attack]: https://www.us-cert.gov/ncas/alerts/TA14-013A
[portsnap]: https://www.freebsd.org/doc/en/books/handbook/ports-using.html
[rc.conf]: https://www.freebsd.org/cgi/man.cgi?rc.conf(5)
