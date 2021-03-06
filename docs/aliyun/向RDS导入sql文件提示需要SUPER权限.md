# 向RDS导入SQL文件提示需要SUPER权限

> https://help.aliyun.com/knowledge_detail/41701.html

往RDS导入SQL文件报错（使用管理员账号 可登陆数据库）：

```
Access denied; you need (at least one of) the SUPER privilege(s) for this operation --常见于RDS MySQL 5.6 
```

查看SQL文件存在GTID

```
# vim jztvxmt_auto.sql
...
SET @@GLOBAL.GTID_PURGED='421589e4-99c8-11e6-bda5-d89d672b3550:1-2112372,
4cf68dcf-cfa4-11e7-a230-008cfae40ca4:1-142118,
53d2a317-99c8-11e6-bda5-d89d672a80ec:1-1539912,
753ea1c9-41be-11e7-84e6-7cd30ac42738:1-1256801,
7db34c2e-3f47-11e8-ba28-6c92bf31607b:1-1809003';
...
```

解决方法：

* 1、直接删除GTID_PURGED语句
* 2、可以尝试使用类似以下命令删除。

```bash
awk '{ if (index($0,"GTID_PURGED")) { getline; while (length($0) > 0) { getline; } } else { print $0 } }' your.sql | grep -iv 'set @@' > your_revised.sql
```

> 可以导出的时候在mysqldump命令后添加参数 `--set-gtid-purged=off`来取消输出GTID_PURGED子句。
