# SQL 语句

SQL语句常见3类：

* DDL(data definition language) 数据定义语言：CREATE ALTER DROP等
* DCL(data control language   ) 数据控制语言：GRANT REVOKE COMMIT ROLLBACK等
* DML(data manipulation language) 数据操作语言；SELECT INSERT DELETE UPDATE等

数据库逻辑包含关系：数据库服务器 - 数据库（不同实例）- 库 - 表 - 行列（数据）

## 数据库

查看数据库

```sql
show databases;

show databases like 'db%';
```

使用/切换数据库

```sql
use database_name
```

创建数据库

```sql
create database db1;

CREATE DATABASE `db1` DEFAULT CHARACTER SET utf8;
```

查看建库语句

```sql
show create database db1\G;
```

删除数据库

```sql
drop database db1;
```

## 数据表

查看数据库中的表

```sql
show tables;
```

创建表

```sql
create table tb1(
    id int(5) not null auto_increment,
    name char(20) not null,
    age tinyint(2) not null default '0',
    dept varchar(16) default null,
    primary key (id)
) engine=innodb default charset=utf8 collate=utf8_general_ci comment='表1';
```

查看表结构

```sql
show create table tb1\G;

desc tb1;
```

更改表名

> rename table 表名 to 新表名

> 或

> alter table 表名 rename to 新表名;

```sql
rename table tb1 to tb2;

alter table tb2 rename to tb1;
```

删除表

```sql
drop table 表名
```

## 字段

修改表字段

添加：alter table 表名 add 字段名 类型 其他(after字段|first)

> 增加多个字段：多个add，用逗号隔开

```sql
alter table tb1 add sex tinyint(2) not null default 0 after id;
```

删除字段：alter table 表名 drop 字段

```sql
alter table tb1 drop sex;
```

改变字段类型：

alter table 表名 modify 列 类型 ...

```sql
alter table tb1 add sex tinyint(2) not null default 0 after name;
alter table tb1 modify sex char(1) after name;
```

```sql
-- 再用modify改回来
alter table tb1 modify sex tinyint(2) not null default 0 after name;
```

修改字段名称

alter table 表名 change 字段 新字段 类型 ...

```sql
alter table tb1 change dept department varchar(16);
```

## CURD 操作

#### 插入数据

1、指定列插入

```sql
insert into tb1(name) values('duliang');
```

2、不指定列，就要按顺序插入

```sql
insert into tb1 values(2,'dl',0,NULL);
```

3、批量插入

```sql
insert into tb1 values(3,'dl2',0,NULL),(4,'dl3',0,NULL),(5,'dl4',0,NULL);
```

#### 修改更新表数据

update 表名 set 字段=新值, where 条件

```sql
update tb1 set name='x', age=18 where id=5;
```

#### 删除表中数据

delete from 表名 where 条件

```sql
delete from tb1 where id=5;
```

## 索引

## 存储过程

## 触发器

## 事件

## 分区
