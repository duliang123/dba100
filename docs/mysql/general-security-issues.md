# 一般安全问题


* 不要让任何人（除了`root`账号）访问 mysql库。
* 了解MySQL访问权限系统的工作原理。 使用GRANT和REVOKE语句来控制对MySQL的访问。 不要授予超出必要的权限。 永远不会授予所有主机权限。
* 试试 ` mysql -u root`。 如果您能够在不被要求输入密码的情况下成功连接到服务器，则任何人都可以以具有完全权限的MySQL root用户身份连接到MySQL服务器！ 查看MySQL安装说明，特别注意有关设置root密码的信息。 
* 使用`SHOW GRANTS`语句检查哪些帐户可以访问什么。 然后使用REVOKE语句删除那些不必要的权限。
* 不要在数据库中存储明文密码。 使用 `SHA2()` 或其他一些单向散列函数并存储散列值来代替。选择一些字符串用作salt，并使用`hash(hash(password)+salt)` 值。
* 不要从词典中选择作为密码，使用复杂度高的字符串组合作为密码。
* 使用防火墙，将MySQL放到防火墙后面或`DMZ`。尝试使用`nmap`等工具从Internet扫描端口。 MySQL默认使用端口3306。 不应从不受信任的主机访问此端口。使用`telnet server_host 3306`命令来检查，如果telnet挂起或连接被拒绝，则端口被阻塞，这就是你想要的。
* 访问MySQL的应用程序不应信任用户输入的任何数据，应使用适当的防御性编程技术编写。
* 不要通过Internet传输普通（未加密）数据。而是使用加密协议，如SSL或SSH。 MySQL支持内部SSL连接。 另一种技术是使用SSH端口转发为通信创建加密（和压缩）隧道。
* 学习使用tcpdump和字符串实用程序。 在大多数情况下，您可以通过发出如下命令来检查MySQL数据流是否未加密：`tcpdump -l -i eth0 -w - src or dst port 3306 | strings`。 提示：如果您没有看到明文数据，这并不总是意味着信息实际上是加密的。 如果您需要高安全性，请咨询安全专家。



## 保持密码安全

* 使用` mysql_config_editor`实用程序，该实用程序使您可以将身份验证凭据存储在名为.mylogin.cnf的加密登录路径文件中。 MySQL客户端程序稍后可以读取该文件以获取用于连接到MySQL服务器的身份验证凭据。
* 在命令行上使用--password = password或-ppassword选项。 例如：`mysql -u francis -pfrank db_name`，这很方便但不安全。
* 在命令行上使用--password或-p选项，但不指定密码值。 在这种情况下，客户端程序以交互方式请求密码：`mysql -u francis -p db_name`。
* 将密码存储在选项文件中。
.my.cnf
```
[client]
password=password
```

为了保证密码的安全，除了您自己之外，任何人都不应该访问该文件。要确保这一点，请将文件访问模式设置为400或600。例如：
```
chmod 600 .my.cnf
```

要从命令行命名包含密码的特定选项文件，请使用--defaults file=file_name选项，其中file_name是文件的完整路径名。例如：

```
mysql --defaults-file=/home/francis/mysql-opts
```

* 不要将密码存储在`MYSQL_PWD`环境变量中。
* 在UNIX上，MySQL客户机将执行语句的记录写入历史文件。默认情况下，此文件名为`.mysql_history`，并在主目录中创建。密码可以在诸如create user和alter user等SQL语句中以纯文本形式写入，因此如果使用这些语句，它们将被记录在历史文件中。要保证此文件的安全，请使用限制性访问模式，与前面对.my.cnf文件所述的方式相同。

```bash
# ll /root/.mysql_history
-rw------- 1 root root 687 Jul 16 16:12 /root/.mysql_history
```


## 密码安全管理员指南

数据库管理员应该使用以下准则来保护密码的安全。

* mysql将用户帐户的密码存储在mysql.user系统表中。不应向任何非管理帐户授予对此表的访问权限。
* 帐户密码可能过期，因此用户必须重置它们。
* `validate_password` 插件可用于对可接受的密码实施策略。
* 应保护可能写入密码的日志文件等文件。
