# MySQL程序 

## MySQL服务器和服务器启动程序

* mysqld — MySQL服务器
* mysqld_safe — MySQL服务器启动脚本
* mysql.server — MySQL服务器启动脚本
* mysqld_multi — 管理多个MySQL实例

## MySQL安装相关程序

* comp_err — Compile MySQL Error Message File
* mysql_install_db — 初始化MySQL数据目录 datadir `从MySQL 5.7.6开始，不推荐使用mysql_install_db，因为它的功能已集成到mysqld中`
* mysql_plugin — 配置MySQL服务器插件
* mysql_secure_installation — 提升MySQL安装安全性
* mysql_ssl_rsa_setup — 创建SSL/RSA文件
* mysql_tzinfo_to_sql — 加载时区表
* mysql_upgrade — 检查并升级MySQL表

## MySQL客户端程序

* mysql — MySQL命令行客户端
* mysqladmin — 管理MySQL服务器的客户端
* mysqlcheck — 表维护程序
* mysqldump — 数据库逻辑备份程序
* mysqlimport — 数据导入程序
* mysqlpump — 数据库备份程序
* mysqlshow — 显示数据库，表和列信息
* mysqlslap — 压力测试工具

## MySQL管理和实用程序

* innochecksum — 离线InnoDB文件校验工具
* myisam_ftdump — 显示全文索引信息
* myisamchk — MyISAM表维护实用程序
* myisamlog — 显示MyISAM日志文件内容
* myisampack — 生成压缩的只读MyISAM表
* mysql_config_editor — MySQL配置实用程序
* mysqlbinlog — 处理二进制日志文件的实用程序
* mysqldumpslow — 总结慢查询日志文件

## MySQL程序开发实用程序

* mysql_config — 显示编译客户端的选项
* my_print_defaults — 从选项文件显示选项
* resolve_stack_dump — Resolve Numeric Stack Trace Dump to Symbols

## 实践

### mysql_config_editor

`mysql_config_editor`程序将身份验证凭据存储在名为`.mylogin.cnf`文件中，MySQL客户端程序稍后可以读取该文件以获取用于连接到MySQL服务器的身份验证凭据。

Linux系统下，`.mylogin.cnf`在用户家目录下生成

```bash
ll /root/.mylogin.cnf
-rw------- 1 root root 24 Aug 17 15:23 /root/.mylogin.cnf
```


**生成**

```bash
# mysql_config_editor set --login-path=login-local-3306 --socket=/data/mysql_3306/tmp/mysql.sock --user=root --password
Enter password:

# mysql_config_editor print --all
[login-local-3306]
user = root
password = *****
socket = /data/mysql_3306/tmp/mysql.sock
```

**使用**

```bash
mysql --login-path=login-local-3306
```

**删除**

```bash
mysql_config_editor remove --login-path=login-local-3306
```


**命令帮助信息**

```bash
# mysql_config_editor --help
mysql_config_editor Ver 1.0 Distrib 5.7.26, for linux-glibc2.12 on x86_64
Copyright (c) 2012, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

MySQL Configuration Utility.
Usage: mysql_config_editor [program options] [command [command options]]
  -#, --debug[=#]     This is a non-debug version. Catch this and exit.
  -?, --help          Display this help and exit.
  -v, --verbose       Write more information.
  -V, --version       Output version information and exit.

Variables (--variable-name=value)
and boolean options {FALSE|TRUE}  Value (after reading options)
--------------------------------- ----------------------------------------
verbose                           FALSE

Where command can be any one of the following :
       set [command options]     Sets user name/password/host name/socket/port
                                 for a given login path (section).
       remove [command options]  Remove a login path from the login file.
       print [command options]   Print all the options for a specified
                                 login path.
       reset [command options]   Deletes the contents of the login file.
       help                      Display this usage/help information.
```

```bash
# mysql_config_editor set --help
mysql_config_editor Ver 1.0 Distrib 5.7.26, for linux-glibc2.12 on x86_64
Copyright (c) 2012, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

MySQL Configuration Utility.

Description: Write a login path to the login file.
Usage: mysql_config_editor [program options] [set [command options]]
  -?, --help          Display this help and exit.
  -h, --host=name     Host name to be entered into the login file.
  -G, --login-path=name
                      Name of the login path to use in the login file. (Default
                      : client)
  -p, --password      Prompt for password to be entered into the login file.
  -u, --user=name     User name to be entered into the login file.
  -S, --socket=name   Socket path to be entered into login file.
  -P, --port=name     Port number to be entered into login file.
  -w, --warn          Warn and ask for confirmation if set command attempts to
                      overwrite an existing login path (enabled by default).
                      (Defaults to on; use --skip-warn to disable.)

Variables (--variable-name=value)
and boolean options {FALSE|TRUE}  Value (after reading options)
--------------------------------- ----------------------------------------
host                              (No default value)
login-path                        client
user                              (No default value)
socket                            (No default value)
port                              (No default value)
warn                              TRUE
```

