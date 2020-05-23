# 用户管理

## 常见用户权限分配

```sql
```

## MySQL访问控制和帐户管理


查看给定帐户拥有哪些特权：

```sql
SHOW GRANTS FOR 'joe'@'office.example.com';
SHOW GRANTS FOR 'joe'@'home.example.com';
```

MySQL提供的权限：

https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html



### 访问控制，第一阶段：连接验证

账户|主机
---|---
'fred'|'h1.example.net'
''	 |  'h1.example.net'
'fred'|'%'
''	  | '%'
'fred'|'%.example.net'
'fred'|'x.example.%'
'fred'|'198.51.100.177'
'fred'|'198.51.100.%'
'fred'|'198.51.100.0/255.255.255.0'


### 访问控制，第2阶段：请求验证

建立连接后，服务器进入访问控制的第2阶段。 对于通过该连接发出的每个请求，服务器确定您要执行的操作，然后检查您是否具有足够的权限来执行此操作。 这是授权表中的特权列发挥作用的地方。 这些权限可以来自任何用户，db，tables_priv，columns_priv或procs_priv表。 

`user`表授予全局权限。

`db`表授予特定于数据库的权限。


### 添加帐户，分配权限和删除帐户

* `CREATE USER` and `DROP USER` 创建和删除帐户
* `GRANT` and `REVOKE` 为帐户分配权限和撤消权限
* `SHOW GRANTS` 显示帐户权限分配

**创建帐户和授予权限**

```sql
CREATE USER 'finley'@'localhost'
  IDENTIFIED BY 'password';
GRANT ALL
  ON *.*
  TO 'finley'@'localhost'
  WITH GRANT OPTION;

CREATE USER 'finley'@'%.example.com'
  IDENTIFIED BY 'password';
GRANT ALL
  ON *.*
  TO 'finley'@'%.example.com'
  WITH GRANT OPTION;

CREATE USER 'admin'@'localhost'
  IDENTIFIED BY 'password';
GRANT RELOAD,PROCESS
  ON *.*
  TO 'admin'@'localhost';

CREATE USER 'dummy'@'localhost';
```

1、两个帐户的用户名为finley。 两者都是具有完全全局权限的超级用户帐户。
> `finley'@'localhost` 账户只能用于本机连接。

> `finley'@'%.example.com'` 账户可以用于连接example.com域中的任何主机。

TODO: 用户表排序顺序

2、`admin'@'localhost`帐户只能由admin用于从本地主机连接。 它被授予全局`RELOAD`和`PROCESS`管理权限。 这些权限使管理员用户能够执行`mysqladmin reload, mysqladmin refresh`和`mysqladmin flush-xxx`命令，以及` mysqladmin processlist `。 没有授予访问任何数据库的权限。 您可以使用GRANT语句添加此类权限。

3、`dummy'@'localhost` 帐户没有密码（这是不安全的，不推荐）。 此帐户只能用于从本地主机进行连接。 没有授予任何权限。


前面的示例在全局级别授予权限。 下一个示例创建三个帐户并授予他们较低级别的访问权限; 也就是说，对于数据库中的特定数据库或对象。 每个帐户的用户名都是自定义的，但主机名部分不同：

```sql
CREATE USER 'custom'@'localhost'
  IDENTIFIED BY 'password';
GRANT ALL
  ON bankaccount.*
  TO 'custom'@'localhost';

CREATE USER 'custom'@'host47.example.com'
  IDENTIFIED BY 'password';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
  ON expenses.*
  TO 'custom'@'host47.example.com';

CREATE USER 'custom'@'%.example.com'
  IDENTIFIED BY 'password';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
  ON customer.addresses
  TO 'custom'@'%.example.com';
```


**WITH GRANT OPTION** 表示该用户可以将自己拥有的权限授权给别人

```sql
-- root权限下
grant  on *.* to 'admin'@'localhost' WITH GRANT OPTION;

-- admin账户下
grant insert on *.* to 'dummy'@'localhost';
```


**检查帐户权限和属性**

```sql
SHOW GRANTS FOR 'admin'@'localhost';
```

```sql
SHOW CREATE USER 'admin'@'localhost'\G
```


**撤销帐户权限**

撤销全局权限

```sql
REVOKE ALL
  ON *.*
  FROM 'finley'@'%.example.com';

REVOKE RELOAD
  ON *.*
  FROM 'admin'@'localhost';
```

撤消数据库级权限

```sql
REVOKE CREATE,DROP
  ON expenses.*
  FROM 'custom'@'host47.example.com';
```

撤消表级权限

```sql
REVOKE INSERT,UPDATE,DELETE
  ON customer.addresses
  FROM 'custom'@'%.example.com';
```

要检查权限撤销的效果，请使用SHOW GRANTS：

```sql
SHOW GRANTS FOR 'admin'@'localhost';
```

**删除账户**

```sql
DROP USER 'finley'@'localhost';
DROP USER 'finley'@'%.example.com';
DROP USER 'admin'@'localhost';
DROP USER 'dummy'@'localhost';
```

**预留帐户**

MySQL安装过程的一部分是数据目录初始化。 在数据目录初始化期间，MySQL会创建应被视为保留的用户帐户：

```sql
mysql> select host,user from mysql.user;
+---------------+---------------+
| host          | user          |
+---------------+---------------+
| localhost     | mysql.session |
| localhost     | mysql.sys     |
| localhost     | root          |
+---------------+---------------+
```

* 'root'@'localhost：用于管理目的。 此帐户具有所有权限，可以执行任何操作。`建议root帐户重命名为其他帐户`。
* 'mysql.sys'@'localhost'：用作sys模式对象的DEFINER。 使用`mysql.sys`帐户可以避免DBA重命名或删除root帐户时出现的问题。 此帐户已锁定，因此无法用于客户端连接。
* 'mysql.session'@'localhost'：插件在内部使用以访问服务器。 此帐户已锁定，因此无法用于客户端连接。

**权限更改什么时候生效**

如果在没有`--skip-grant-tables`选项的情况下启动`mysqld`服务器，它会在启动序列期间将所有授权表内容读入内存。内存表在此时对访问控制有效。

如果使用帐户管理语句间接修改授权表，服务器会注意到这些更改并立即再次将授权表加载到内存中。示例包括 `GRANT, REVOKE, SET PASSWORD, and RENAME USER`。

如果直接使用 `INSERT, UPDATE, or DELETE`等语句（不推荐）修改授权表，则更改对权限检查没有影响，直到您告诉服务器重新加载表或重新启动它。 因此，如果直接更改授权表但忘记重新加载它们，则在重新启动服务器之前，更改不会生效。 这可能会让您想知道为什么您的更改似乎没有任何区别！

```
要告诉服务器重新加载授权表，请执行flush-privileges操作。
这可以通过发出 `FLUSH PRIVILEGES` 语句或执行 `mysqladmin flush-privileges` 或 `mysqladmin reload` 命令来完成。
```

授权表重新加载会影响每个现有客户端会话的权限，如下所示：

* 表和列权限更改将随客户端的下一个请求生效。
* 数据库权限更改将在客户端下次执行 `USE db_name` 语句时生效。
* 对于已连接的客户端，全局权限和密码不受影响。 这些更改仅在后续连接的会话中生效。

如果使用 `--skip-grant-tables` 选项启动服务器，则它不会读取授权表或实现任何访问控制。 任何用户都可以连接并执行任何操作，这是不安全的。 为了使服务器因此开始读取表并启用访问检查，请刷新权限。


**分配帐户密码**

MySQL将凭证存储在 `mysql` 系统数据库的 `user` 表中，分配或修改密码的操作仅允许具有 `CREATE USER` 权限的用户，或者 `mysql` 数据库的权限（创建新帐户的`INSERT`权限，修改现有帐户的`UPDATE`权限）。

如果启用了`read_only`系统变量，则使用帐户修改语句（如 CREATE USER 或 ALTER USER）还需要` SUPER`权限。

* 要在创建新帐户时指定密码，请使用`CREATE USER`并包含`IDENTIFIED BY`子句：

```sql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';
```

* 要为现有帐户分配或更改密码，请使用带有`IDENTIFIED BY`子句的`ALTER USER`语句：

```sql
ALTER USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';
```

* 如果您未以匿名用户身份进行连接，则可以更改自己的密码，而无需按字面指定自己的帐户：

```sql
ALTER USER USER() IDENTIFIED BY 'password';
```

* 要从命令行更改帐户密码，请使用`mysqladmin`命令：

```sql
mysqladmin -u user_name -h host_name password "password"
```

> 使用`mysqladmin`设置密码应该被认为是不安全的。 在某些系统上，您的密码对系统状态程序（例如`ps`）可见，其他用户可以调用它来显示命令行。

此命令设置密码的帐户是`mysql.user`系统表中的一行，该行与`User`列中的`user_name`和您在 `Host` 列中连接的客户端主机匹配。


**密码管理**

MySQL使数据库管理员可以手动设置帐户密码过期，并建立自动密码过期的策略。 

```sql
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE;
```

此操作将密码在相应的 `mysql.user` 系统表行中标记为已过期。

要在全局范围内建立自动密码过期策略，使用`default_password_lifetime`。 其默认值为`0`，禁用自动密码到期。 如果`default_password_lifetime`的值是正整数N，则表示允许的密码生存期，因此密码必须每N天更改一次。

> 注意：

> 在5.7.11之前，默认的`default_password_lifetime`值为360（密码每年大约必须更改一次）。 对于此类版本，请注意，如果您未对default_password_lifetime变量或单个用户帐户进行任何更改，则每个用户密码将在360天后过期，并且帐户将以受限模式运行。 使用该帐户连接到服务器的客户端会收到错误，指示必须更改密码：ERROR 1820（HY000）：在执行此语句之前，必须使用ALTER USER语句重置密码。

> 但是，对于自动连接到服务器的客户端（例如通过脚本建立的连接），很容易错过。 为避免此类客户端因密码过期而突然停止工作，请确保更改这些客户端的密码过期设置，如下所示：`ALTER USER 'script'@'localhost' PASSWORD EXPIRE NEVER`；或者，将` default_password_lifetime 变量设置为 0,`，从而禁用所有用户的自动密码到期。

* 在`my.cnf`配置文件中定义

```
# 六个月过期
[mysqld]
default_password_lifetime=180

# 或者永不过期
[mysqld]
default_password_lifetime=0
```

* 也可以在运行时更改

```sql
SET GLOBAL default_password_lifetime = 180;

-- 或者

SET GLOBAL default_password_lifetime = 0;
```

全局密码到期策略适用于尚未设置为覆盖它的所有帐户。 要为个人帐户建立策略，请使用`CREATE USE`和A`ALTER USER`语句的`PASSWORD EXPIRE`选项。

```sql
CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;

CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE NEVER;
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE NEVER;


-- 遵循该语句指定的所有帐户的全局过期策略 这里的DEFAULT的值是参照default_password_lifetime
CREATE USER 'jeffrey'@'localhost' PASSWORD EXPIRE DEFAULT;
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE DEFAULT;
```

```
mysql> select host,user,plugin,password_expired,password_last_changed,password_lifetime,account_locked from  mysql.user;
+---------------+---------------+-----------------------+------------------+-----------------------+-------------------+----------------+
| host          | user          | plugin                | password_expired | password_last_changed | password_lifetime | account_locked |
+---------------+---------------+-----------------------+------------------+-----------------------+-------------------+----------------+
| localhost     | root          | mysql_native_password | N                | 2019-07-16 13:58:15   |              NULL | N              |
| localhost     | mysql.session | mysql_native_password | N                | 2019-07-16 12:12:22   |              NULL | Y              |
| localhost     | mysql.sys     | mysql_native_password | N                | 2019-07-16 12:12:22   |              NULL | Y              |
+---------------+---------------+-----------------------+------------------+-----------------------+-------------------+----------------+

```

**设置帐户资源限制**

限制客户端使用MySQL服务器资源的一种方法是将全局`max_user_connections`系统变量设置为非零值。这限制了任何给定帐户可以同时进行的连接数，但对连接后客户端可以执行的操作没有限制。此外，设置 `max_user_connections` 无法管理个人帐户。

为了解决这些问题，MySQL允许使用这些服务器资源限制个人帐户：

* 帐户每小时可以发出的查询数
* 帐户每小时可以发布的更新次数
* 帐户每小时可以连接到服务器的次数
* 帐户与服务器同时连接的数量

要在帐户创建时为帐户建立资源限制，请使用`CREATE USER`语句。 要修改现有帐户的限制，请使用`ALTER USER`。提供一个WITH子句，命名每个要限制的资源。 每个限制的默认值为零（无限制）。 例如，要创建可以访问客户数据库但只能以有限方式访问的新帐户，请发出以下语句：

```sql
mysql> CREATE USER 'francis'@'localhost' IDENTIFIED BY 'frank'
    ->     WITH MAX_QUERIES_PER_HOUR 20
    ->          MAX_UPDATES_PER_HOUR 10
    ->          MAX_CONNECTIONS_PER_HOUR 5
    ->          MAX_USER_CONNECTIONS 2;
```

对于 `MAX_USER_CONNECTIONS`，限制是一个整数，表示帐户的最大同时连接数。如果此限制设置为零，则全局 `max_user_connections` 系统变量值确定同时连接的数量。如果`max_user_connections`也为零，则帐户没有限制。

要修改现有帐户的限制，请使用 ALTER USER语句。以下语句将查询限制更改为francis 100：

```sql
mysql> ALTER USER 'francis'@'localhost' WITH MAX_QUERIES_PER_HOUR 100;
```

要删除限制，请将其值设置为零。例如，要删除每小时francis 可以连接的次数限制，请使用以下语句：

```sql
mysql> ALTER USER 'francis'@'localhost' WITH MAX_CONNECTIONS_PER_HOUR 0;
```

服务器在与该帐户对应的 `user` 表行中存储帐户的资源限制 。`max_questions`，`max_updates`和 `max_connections` 列存储每小时限制和`max_user_connections`列存储 MAX_USER_CONNECTIONS 限制。

```sql
select max_questions,max_updates,max_connections,max_user_connections from mysql.user;
```


> 当任何帐户对其使用任何资源具有非零限制时，将进行资源使用计数。

可以为所有帐户全局重置当前每小时资源使用计数，也可以针对给定帐户单独重置当前每小时资源使用计数：

* 要将所有帐户的当前计数重置为零，请发出 ` FLUSH USER_RESOURCES` 声明。还可以通过重新加载授权表来重置计数（例如，使用`FLUSH PRIVILEGES`语句或`mysqladmin reload`命令）。
* 给定帐户单独重置：TODO

每小时计数器重置不会影响MAX_USER_CONNECTIONS限制。

服务器启动时，所有计数从零开始。 计数不会通过服务器重启而延续。


**USER()、CURRENT_USER()**

* USER() 返回当前登陆的MySQL用户名和主机名。
* CURRENT_USER() 可以调用`CURRENT_USER()`来确定客户端登陆是哪个帐户，其值由帐户的`user`表行的`User`和`Host`列构成。
