# MySQL 8

> 文档：https://dev.mysql.com/doc/refman/8.0/en/


## 安装

安装要求

* CMake
* make 
* GCC 5.3
* ncurses 库

编译安装

```bash
# Preconfiguration setup
groupadd mysql
useradd -r -g mysql -s /bin/false mysql

# Beginning of source-build specific instructions
tar zxvf mysql-VERSION.tar.gz
cd mysql-VERSION
mkdir bld
cd bld
cmake ..
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
