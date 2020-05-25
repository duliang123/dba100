# 获得MySQL帮助

获得帮助的途径

* 一、官方文档
* 二、mysql客户端工具
* 三、? show 命令（查看一些数据库创建的DDL，事件的DDL）

## 一、官方文档
http://dev.mysql.com/doc/refman/5.7/en/


## 二、mysql客户端工具

```sql
# mysql -uroot -p -S /data/mysql_3306/mysql.sock
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.6.29-log Source distribution
mysql> help //输入hel 或者 ? 就显示客户端的帮助，而不是服务器端。

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.

clear     (\c) Clear the current input statement.
#清除当前的sql输入语句，如输错sql语句（未加分号时）回车，可跳出当前的输入。当然也可以在语句后输入分号结束，但有报错信息。

connect   (\r) Reconnect to the server. Optional arguments are db and host.
#mysql的连接被服务器复位了，没必要exit重新连。通过 connect 数据库名 主机名 连接即可

delimiter (\d) Set statement delimiter.
#修改执行符号，通常是“分号”，写存储过程时有很多sql需要分号分隔开，但mysql检测到分号会执行该sql语句，存储过程就写不完了。所以delimiter就解决这个问题，改变当然执行的符号。

edit      (\e) Edit command with $EDITOR.
#修改SQL语句，如写了很长（而且有换行）前面有错要修改，会自动调用VI。wq保存退出加 ; 回车就可以执行了

ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.

pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
#查询很多数据的表，让我们控制翻屏显示。
如mysql> pager less；select * from information_schema.tables\G;

print     (\p) Print current command.

prompt    (\R) Change your mysql prompt.
#修改mysql的提示符 可以在my.cnf中的[mysql]章节中定义

quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.

source    (\.) Execute an SQL script file. Takes a file name as an argument.
#从外部执行一个sql脚本

status    (\s) Get status information from the server.
#获得mysql服务器的状态信息（当前连接id,用户、数据库、字符集、启动时间等）

system    (\!) Execute a system shell command.
#后跟操作系统命令，显示系统操作的结果，这样就不用退出mysql再执行系统命令了。

tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
#把查询结果输入到给定文件中去

use       (\u) Use another database. Takes database name as argument.
#切换当前使用的数据库

charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
#改变字符集 如：charset gbk  (改变客户端和连接的字符集，可用status查看)

warnings  (\W) Show warnings after every statement.
#在每一个语句后显示警告信息。

nowarning (\w) Don't show warnings after every statement.
#不显示警告信息

For server side help, type 'help contents'
#查看服务器端帮助，使用help contents
```

1、prompt示例

```sql
mysql> prompt #
PROMPT set to '#'
#select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
#
#prompt mysql>
PROMPT set to 'mysql>'
mysql>
mysql> prompt [\h:] \u@mysql \d \R:\m:\s > 
PROMPT set to '[\h:] \u@mysql \d \R:\m:\s > '
[localhost:] root@mysql (none) 20:02:09 > use test
Database changed
[localhost:] root@mysql test 20:02:15 > show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)
```

2、delimiter示例

```sql
mysql> delimiter $$
mysql> show databases;
    -> $$
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)
```

3、status示例

```sql
mysql> status
--------------
mysql  Ver 14.14 Distrib 5.6.29, for Linux (x86_64) using  EditLine wrapper

Connection id:          20
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         5.6.29-log Source distribution
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:            /data/mysql_3306/mysql.sock
Uptime:                 1 hour 29 min 15 sec

Threads: 1  Questions: 117  Slow queries: 5  Opens: 87  Flush tables: 1  Open tables: 80  Queries per second avg: 0.021
--------------

mysql> 
```

4、tee示例

```sql
mysql> \! > /tmp/tee.txt
mysql> tee /tmp/tee.txt
Logging to file '/tmp/tee.txt'
mysql> use test;
Database changed
mysql> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)

mysql> \! cat /tmp/tee.txt
mysql> \! > /tmp/tee.txt
mysql> tee /tmp/tee.txt
mysql> use test;
Database changed
mysql> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```

5、查看服务器端帮助

```sql
mysql> help contents
You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Help Metadata
   Language Structure
   Plugins
   Procedures
   Storage Engines
   Table Maintenance
   Transactions
   User-Defined Functions
   Utility

mysql> help administration
You asked for help about help category: "Administration"
For more information, type 'help <item>', where <item> is one of the following
topics:
   BINLOG
   CACHE INDEX
   FLUSH
   FLUSH QUERY CACHE
   HELP COMMAND
   KILL
   LOAD INDEX
   RESET
   SET
   SHOW
   SHOW AUTHORS
   SHOW BINARY LOGS
   SHOW BINLOG EVENTS
   SHOW CHARACTER SET
   SHOW COLLATION
   SHOW COLUMNS
   SHOW CONTRIBUTORS
   SHOW CREATE DATABASE
   SHOW CREATE EVENT
   SHOW CREATE FUNCTION
   SHOW CREATE PROCEDURE
   SHOW CREATE TABLE
   SHOW CREATE TRIGGER
   SHOW CREATE VIEW
   SHOW DATABASES
   SHOW ENGINE
   SHOW ENGINES
   SHOW ERRORS
   SHOW EVENTS
   SHOW FUNCTION CODE
   SHOW FUNCTION STATUS
   SHOW GRANTS
   SHOW INDEX
   SHOW MASTER STATUS
   SHOW OPEN TABLES
   SHOW PLUGINS
   SHOW PRIVILEGES
   SHOW PROCEDURE CODE
   SHOW PROCEDURE STATUS
   SHOW PROCESSLIST
   SHOW PROFILE
   SHOW PROFILES
   SHOW RELAYLOG EVENTS
   SHOW SLAVE HOSTS
   SHOW SLAVE STATUS
   SHOW STATUS
   SHOW TABLE STATUS
   SHOW TABLES
   SHOW TRIGGERS
   SHOW VARIABLES
   SHOW WARNINGS
   ```
   
