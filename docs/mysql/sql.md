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

#### 主键索引

```sql
-- 创建主键索引
alter table tb1 change id id int primary key auto_increment;

-- 删除主键索引(主键索引列设置自增auto_increment 则无法删除)
alter table tb1 drop primary key;
```

#### 普通索引

```sql
-- 创建普通索引
alter table tb1 add index index_dept(dept);

-- 建表时：
...,
KEY index_name (name)
);
```

#### 根据列前n个字符创建索引

```sql
create index index_dept on tb1(dept(8));
```

#### 根据多个列创建联合索引

```sql
create index ind_name_dept on tb1(name, dept);
```

#### 根据多个列前n个字符创建联合索引

```sql
create index ind_name_dept on tb1(name(8), dept(10));
```

#### 创建唯一索引

```sql
create unique index uni_ind_name on tb1(name);
```

#### 删除普通索引与唯一索引
```
alter table tb1 drop index index_dept;

drop index index_dept on tb1;
```

#### 查看唯一值数量

```sql
select count(distinct user) from mysql.user;
```

#### 总结

```
主键索引列要求所有内容必须唯一，而普通索引不要求内容必须唯一

1、索引类似书箱目录，加快查询数据速度；
2、要在表的列（字段）上创建索引；
3、索引会加快查询速度，但也会影响更新的速度，因为更新要维护索引数据；
4、索引列并不是越多越好，要在频繁查询的表语句where 后的条件列上建立索引；
5、小表、重复值很多的列上不建立索引，要在大表以及重复值少的条件列上创建索引；
6、多个列联合索引有前缀生效特性；
7、当字段内容前N个字符已经接近唯一时，可以对字段的前N个字符创建索引；
8、索引从工作方式区分：有主键、唯一、普通索引；
9、索引类型会有BTREE(默认)和HASH(适合做缓存 内存数据库)等；
10、数据越多，索引建立时间越慢，影响业务，业务低峰时再建立；
11、尽量在唯一值多的大表上建立索引（姓别、状态列等多重复值列建立意义不大）。
```

```
联合索引有前缀生效特性：
index(a, b, c)联合索引，仅a, ab, abc三个查询条件可以走索引；
b, bc, ac, c等查询条件无法使用索引。

desc 表时Key类型：
PRI 主键索引
MUL 普通索引
UNI 唯一索引（较少用）
```

show index方法查看索引

```
mysql> show index from tb1\G;
*************************** 1. row ***************************
        Table: tb1
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
*************************** 2. row ***************************
        Table: tb1
   Non_unique: 1
     Key_name: idx_name
 Seq_in_index: 1
  Column_name: name
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
2 rows in set (0.00 sec)
```

使用explain查看SQL语句执行计划：查看是否使用到索引 key

```sql 
mysql> explain select * from tb1 where name='duliang'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
         type: ref
possible_keys: idx_name   可能使用
          key: idx_name   实际使用
      key_len: 60
          ref: const
         rows: 1          查了1行
        Extra: Using index condition
1 row in set (0.00 sec)
```


使用explain命令优化SQL语句基本流程：

```
1、查看慢查询SQL语句（连续执行2次 2秒内查看是否有同一条语句在执行）
mysql -uroot -pDL2016-1q2w3e@_@db.com -S /data/mysql_3306/mysql.sock -e 'show full processlist' | egrep -vi 'sleep'

2、explain检查索引执行情况
explain select * from tb1 where name='duliang'\G;

3、切割慢查询日志
mv /data/mysql_3306/mysql-slow.log /data/log/$(date +%F)_slow.log
mysqladmin -uroot -pDL2016-1q2w3e@_@db.com -S /data/mysql_3306/mysql.sock flush-logs

4、mysqlsla 分析日志
```

利用存储过程插入10万条数据测试

```sql
create table test(id int(11) primary key auto_increment, num int(11) not null);

create procedure proc_test(in para int(11))
    begin
        declare i int default 0;
        declare rand_num int;
        while i < para do
            select cast(rand()*10000 as unsigned) into rand_num;
            insert into test(num) values(rand_num);
            set i=i+1;
        end while;
    end;
call proc_test(100000);

select SQL_NO_CACHE * from test;

测试:
...
| 494050 | 5519 |
+--------+------+
49 rows in set (0.33 sec)
mysql> select sql_no_cache * from test where num=5519;   #用了0.33秒

mysql> explain select sql_no_cache * from test where num=5519\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: test
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 485331   扫描了48万行
        Extra: Using where
1 row in set (0.00 sec)

mysql> alter table test add index idx_num(num);   #添加索引
Query OK, 0 rows affected (3.09 sec)
Records: 0  Duplicates: 0  Warnings: 0

...
| 484690 | 5519 |
| 494050 | 5519 |
+--------+------+
49 rows in set (0.00 sec)
mysql> select sql_no_cache * from test where num=5519;   #使用索引只有0秒

mysql> explain select sql_no_cache * from test where num=5519\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: test
         type: ref
possible_keys: idx_num
          key: idx_num
      key_len: 4
          ref: const
         rows: 49   只扫描了49行
        Extra: Using index
1 row in set (0.00 sec)
```

使用profile优化SQL语句

```sql
1、查看是否开启profile功能
mysql> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           0 |
+-------------+
1 row in set, 1 warning (0.00 sec)

2、开启profile
mysql> set profiling = 1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select @@profiling;
+-------------+
| @@profiling |
+-------------+
|           1 |
+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> show profiles;
+----------+------------+--------------------+
| Query_ID | Duration   | Query              |
+----------+------------+--------------------+
|        1 | 0.00018150 | select @@profiling |
+----------+------------+--------------------+
1 row in set, 1 warning (0.00 sec)

3、查询测试
删除上方建立的索引 drop index idx_num on test;
查询：
...
| 484690 | 5519 |
| 494050 | 5519 |
+--------+------+
49 rows in set (0.33 sec)
mysql> show profiles;
+----------+------------+------------------------------------------------+
| Query_ID | Duration   | Query                                          |
+----------+------------+------------------------------------------------+
|        1 | 0.00018150 | select @@profiling                             |
|        2 | 0.33758325 | select sql_no_cache * from test where num=5519 |
+----------+------------+------------------------------------------------+
2 rows in set, 1 warning (0.00 sec)

mysql> show profile for query 2;   #2为上方的Query_ID
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000073 |
| checking permissions | 0.000009 |
| Opening tables       | 0.000064 |
| init                 | 0.000036 |
| System lock          | 0.000013 |
| optimizing           | 0.000016 |
| statistics           | 0.000026 |
| preparing            | 0.000020 |
| executing            | 0.000004 |
| Sending data         | 0.336426 |
| end                  | 0.000018 |
| query end            | 0.000011 |
| closing tables       | 0.000019 |
| freeing items        | 0.000387 |
| logging slow query   | 0.000440 |
| cleaning up          | 0.000023 |
+----------------------+----------+
16 rows in set, 1 warning (0.00 sec)
查看哪个部分占用时间较长

更详细分析:
mysql> show profile cpu, block io, memory, swaps for query 2;
+----------------------+----------+----------+------------+--------------+---------------+-------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out | Swaps |
+----------------------+----------+----------+------------+--------------+---------------+-------+
| starting             | 0.000073 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| checking permissions | 0.000009 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| Opening tables       | 0.000064 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| init                 | 0.000036 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| System lock          | 0.000013 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| optimizing           | 0.000016 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| statistics           | 0.000026 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| preparing            | 0.000020 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| executing            | 0.000004 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| Sending data         | 0.336426 | 0.335949 |   0.000000 |            0 |             0 |     0 |
| end                  | 0.000018 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| query end            | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| closing tables       | 0.000019 | 0.000000 |   0.000000 |            0 |             0 |     0 |
| freeing items        | 0.000387 | 0.001000 |   0.000000 |            0 |             0 |     0 |
| logging slow query   | 0.000440 | 0.000000 |   0.000000 |            0 |             8 |     0 |
| cleaning up          | 0.000023 | 0.000000 |   0.000000 |            0 |             0 |     0 |
+----------------------+----------+----------+------------+--------------+---------------+-------+
16 rows in set, 1 warning (0.00 sec)
```

## 存储过程

## 触发器

## 事件

## 分区
