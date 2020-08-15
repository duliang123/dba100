# MySQL 8

> 文档：https://dev.mysql.com/doc/refman/8.0/en/


## 二进制方式安装

https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html

```
strings /lib64/libc.so.6 | grep ^GLIBC
dnf install libaio numactl
cd /usr/local/src/
mkdir -p /data/mysql_3306/{data,binlog,logs,relay-log,tmp,undolog}
chown -R mysql:mysql /data/mysql_3306

groupadd mysql
useradd -r -g mysql -s /bin/false mysql
tar xvf mysql-8.0.21-linux-glibc2.17-x86_64-minimal.tar.xz
mv mysql-8.0.21-linux-glibc2.17-x86_64-minimal /usr/local

cd /usr/local
ln -s mysql-8.0.21-linux-glibc2.17-x86_64-minimal mysql
cd mysql
mkdir mysql-files
chown mysql:mysql mysql-files
chmod 750 mysql-files
bin/mysqld --defaults-file=/data/mysql_3306/my.cnf --initialize --user=mysql
bin/mysql_ssl_rsa_setup
bin/mysqld_safe --defaults-file=/data/mysql_3306/my.cnf --user=mysql &

# Next command is optional
cp support-files/mysql.server /etc/init.d/mysql.server

cat /data/mysql_3306/data/error.log |grep password
export PATH=$PATH:/usr/local/mysql/bin
```

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'RXDyFF12et3Bgi';
```

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


## 库 用户

```sqk
create database cloudadmin DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

CREATE USER 'cloudadmin'@'127.0.0.1'
  IDENTIFIED BY 'RXDyFF12et3Bgi';
GRANT ALL
  ON cloudadmin.*
  TO 'cloudadmin'@'127.0.0.1'
  WITH GRANT OPTION;
```
