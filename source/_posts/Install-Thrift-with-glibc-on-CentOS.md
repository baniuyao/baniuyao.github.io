title: Install Thrift with glibc on CentOS
date: 2015-01-13 16:08:12
tags:
- c
- thrift
- linux
---

# Install glib

## Install Development Tools

`Development Tools` is a group of pacakges in yum such as `automake`, `autoconf` and so on.

``` bash
sudo yum -y update
sudo yum -y groupinstall "Development Tools"
sudo yum install -y wget
```

## Upgrade some tools

Although some tools are included in the `Development Tools`, `glibc` and `thrift` need new version of those tools. We need to upgrade them.

### autoconf

``` bash
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz
tar xvf autoconf-2.69.tar.gz
cd autoconf-2.69
./configure
make
sudo make install
```

### automake

``` bash
wget http://ftp.gnu.org/gnu/automake/automake-1.14.tar.gz
tar xvf automake-1.14.tar.gz
cd automake-1.14
./configure
make
sudo make install
```

### bison

``` bash
wget http://ftp.gnu.org/gnu/bison/bison-2.5.1.tar.gz
tar xvf bison-2.5.1.tar.gz
cd bison-2.5.1
./configure
make
sudo make install
```

## Install glib (finally)

``` bash
wget http://ftp.gnome.org/pub/gnome/sources/glib/2.42/glib-2.42.0.tar.xz
tar xvf glib-2.42.0.tar.xz
cd glib-2.42.0
./configure
make
sudo make install
```

## Check

`pkgconfig` is in `/usr/local/lib/pkgconfig` or `/usr/lib64/pkgconfig`. Please find it by yourself by `locate pkgconfig`. Further, you must add `PKG_CONFIG_PATH` in your environment variables.

``` bash
ls -lrt /usr/local/lib/pkgconfig
echo "export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig" >> /etc/profile
source /etc/profile
```

# Install Thrift (finally * 2)

If you get `thrift` from `git`, you must run `bootstrap.sh` first.

``` bash
cd thrift-0.9.2
./configure --with-lua=no
result:
Building C++ Library ......... : no
Building C (GLib) Library .... : yes
Building Java Library ........ : no
Building C# Library .......... : no
Building Python Library ...... : yes
Building Ruby Library ........ : no
Building Haskell Library ..... : no
Building Perl Library ........ : no
Building PHP Library ......... : no
Building Erlang Library ...... : no
Building Go Library .......... : no
Building D Library ........... : no
Building NodeJS Library ...... : no
Building Lua Library ......... : yes
make 
make install
```
