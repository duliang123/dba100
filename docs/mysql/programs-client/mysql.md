# 连接登陆

连接方式

* SOCKET `-S/--socket=/tmp/mysql.sock`
* TCP/IP `-h x.x.x.x -P 3306`

```bash
export PATH=$PATH:/usr/local/mysql/bin
ln -s /dev/null $HOME/.mysql_history

mysql -u root -p 连接方式
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
