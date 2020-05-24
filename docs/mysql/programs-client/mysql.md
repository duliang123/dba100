# 连接登陆

## 连接方式

* SOCKET `-S/--socket=/tmp/mysql.sock`
* TCP/IP `-h x.x.x.x -P 3306`

```bash
export PATH=$PATH:/usr/local/mysql/bin
ln -s /dev/null $HOME/.mysql_history
```

```bash
mysql -u root -p 连接方式
```

### 注意

mysql 命令行如果`未指定主机`或`主机为 localhost`，则会发生与本地主机的连接，在类Unix上，默认使用Unix套接字文件进行连接。

我们把 /etc/my.cnf 配置文件中 [mysqld] 内的套接字地址改下：socket = /tmp/1mysql.sock

```
mysql
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
或
mysql -h localhost -u root -p
Enter password: 
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
```

mysql提示找不到默认的/tmp/mysql.sock文件了，加-S参数指定下套接字文件就可以连接了。

```bash
mysql -S /tmp/1mysql.sock
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
