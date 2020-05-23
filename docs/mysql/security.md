# 安装之后安全设置 

查看用户

```sql
select user,host from mysql.user;
```

删除空用户

```sql
delete from mysql.user where user='';
```

修改密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '自定义密码';
```

`MySQL 5.6`

```sql
UPDATE mysql.user SET Password = PASSWORD('自定义密码') WHERE User = 'root';
FLUSH PRIVILEGES;
```
