# 配置和状态信息

* https://dev.mysql.com/doc/refman/5.7/en/server-configuration.html
* [服务器选项，系统变量和状态变量参考](https://dev.mysql.com/doc/refman/5.7/en/server-option-variable-reference.html)

MySQL服务器有许多操作参数，您可以使用命令行选项或配置文件（选项文件）在服务器启动时更改这些参数。 也可以在运行时更改许多参数。

默认选项以给定顺序从以下文件中读取：`/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf`

`mysqld` 从 [mysqld] and [server] 组读取选项 

`mysqld_safe` 从 [mysqld], [server], [mysqld_safe], and [safe_mysqld] 组读取选项 

`mysql.server` 从 [mysqld] and [mysql.server] 组读取选项


MySQL服务器mysqld有许多命令选项和系统变量，可以在启动时设置它们以配置其操作。要确定服务器使用的缺省命令选项和系统变量值，请执行以下命令：

```bash
# 该命令生成所有mysqld选项和可配置系统变量的列表。
shell> mysqld --verbose --help
```


要查看服务器在运行时实际使用的当前系统变量值，请连接到它并执行以下语句：

```sql
SHOW VARIABLES;
```

要查看正在运行的服务器的某些统计和状态指示器，请执行以下语句：

```sql
SHOW STATUS;
```

系统变量和状态信息也可以使用mysqladmin命令查看：

```sql
shell> mysqladmin variables
shell> mysqladmin extended-status
```


MySQL使用非常可扩展的算法，因此您通常可以使用非常少的内存运行。 但是，通常会给MySQL带来更多内存，从而提高性能。

在调优MySQL服务器时，要配置的两个最重要的变量是 `key_buffer_size` 和 `table_open_cache`。 在尝试更改任何其他变量之前，您应首先确信您已正确设置这些设置。

> 如果要对远大于可用内存的表执行GROUP BY或ORDER BY操作，请增加 read_rnd_buffer_size 的值以加快排序操作后的行读取速度。
