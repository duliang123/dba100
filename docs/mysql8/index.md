# MySQL 8

> 文档：https://dev.mysql.com/doc/refman/8.0/en/


## 二进制方式安装

https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html


## 编译方式安装

安装要求

* CMake
* make 
* GCC 5.3
* ncurses 库

编译参数

https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html

编译安装

```bash
VERSION = 8.0.21
cd /usr/local/src
wget -c https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-boost-${VERSION}.tar.gz
tar zxvf mysql-boost-${VERSION}.tar.gz 
```

```bash
# Preconfiguration setup
groupadd mysql
useradd -r -g mysql -s /bin/false mysql

# Beginning of source-build specific instructions
tar zxvf mysql-VERSION.tar.gz
cd mysql-VERSION
mkdir bld
cd bld

cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
      -DSYSCONFDIR=/etc \
      -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
      -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
      -DDEFAULT_CHARSET=utf8mb4 \
      -DDEFAULT_COLLATION=utf8mb4_general_ci \
      -DENABLED_LOCAL_INFILE=1 \
      -DWITH_BOOST=../boost

make
make install
# End of source-build specific instructions

# Postinstallation setup
cd /usr/local/mysql
mkdir mysql-files
chown mysql:mysql mysql-files
chmod 750 mysql-files
bin/mysqld --initialize --user=mysql
#bin/mysql_ssl_rsa_setup
bin/mysqld_safe --user=mysql &

# Next command is optional
support-files/mysql.server /etc/init.d/mysql.server
```
