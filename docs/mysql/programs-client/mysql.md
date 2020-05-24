# 连接登陆

连接方式

* SOCKET `-S/--socket=/tmp/mysql.sock`
* TCP/IP `-h x.x.x.x -P 3306`

```bash
export PATH=$PATH:/usr/local/mysql/bin
ln -s /dev/null $HOME/.mysql_history

mysql -u root -p 连接方式
```

mysql 命令行不加参数，默认使用Unix套接字文件进行连接

> 我们把/etc/my.cnf配置文件中[mysqld]内的套接字地址改下：socket = /tmp/1mysql.sock

```
# mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

mysql提示找不到默认的/tmp/mysql.sock文件了
```

## 客户端定制配置

~/.my.cnf

* [client] 所有客户端工具有效
* [mysql]  mysql 这个命令行工具有效

```
[client]
port=3306
socket=/tmp/mysql.sock

[mysql]
port=3306
socket=/tmp/mysql.sock

no-auto-rehash
connect_timeout=2
```
