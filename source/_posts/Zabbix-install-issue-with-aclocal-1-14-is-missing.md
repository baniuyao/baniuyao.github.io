title: Zabbix install issues with aclocal 1.14 is missing
date: 2015-02-27 07:14:35
tags:
- Zabbix
- C
---

# Backgroud

`configure` is okay, the next step is `make install`. But `make install` exist with error which said it cannot find `aclocal-1.14` like below:

``` bash
[baniuyao@YaoRenjies-CentOS zabbix-2.4.4]$ sudo make install
CDPATH="${ZSH_VERSION+.}:" && cd . && /bin/sh /home/baniuyao/apps/zabbix-2.4.4/missing aclocal-1.14 -I m4
/home/baniuyao/apps/zabbix-2.4.4/missing: line 81: aclocal-1.14: command not found
WARNING: 'aclocal-1.14' is missing on your system.
         You should only need it if you modified 'acinclude.m4' or
         'configure.ac' or m4 files included by 'configure.ac'.
         The 'aclocal' program is part of the GNU Automake package:
         <http://www.gnu.org/software/automake>
         It also requires GNU Autoconf, GNU m4 and Perl in order to run:
         <http://www.gnu.org/software/autoconf>
         <http://www.gnu.org/software/m4/>
         <http://www.perl.org/>
make: *** [aclocal.m4] Error 127
```

But the system has `aclocal-1.14` already:

``` bash
[baniuyao@YaoRenjies-CentOS ~]$ which aclocal-1.14
/usr/local/bin/aclocal-1.14
```

# Solution

This is because we don’t have some `autoconf`(*.ac) and `automake`(*.am) files. We can `touch` these.

``` bash
touch configure.ac aclocal.m4 configure Makefile.am Makefile.in
```

Or we can use `autoreconf` to generate.

``` bash
sudo autoreconf -ivf
```

Let’s figure out what `-ivf` means.

``` bash
-v, --verbose            verbosely report processing
-f, --force              consider all files obsolete
-i, --install            copy missing auxiliary files
```
