# mysqldump

本节介绍如何使用mysqldump生成转储文件，以及如何重新加载转储文件。 转储文件使用场景：

* 作为备份，在数据丢失的情况下启用数据恢复。
* 作为设置从库复制的数据源。
* 作为实验数据的来源：
    * 制作可以在不更改原始数据的情况下使用的数据库副本。
    * 测试潜在的升级不兼容性。

mysqldump优缺点：

> 逻辑备份，效率不是特别高，不适用大数据备份（超50G）

> 简单、方便、可靠、移植性好（与存储引擎无关，因为是从MySQL服务器中提取数据而生成的，所以消除了底层数据存储的不同）



mysqldump产生两种类型的输出，具体取决于是否给出了`--tab`选项：

* 没有--tab

> mysqldump会将SQL语句写入标准输出。 此输出包含用于创建转储对象（数据库，表，存储例程等）的CREATE语句，以及用于将数据加载到表中的INSERT语句。 输出可以保存在文件中，稍后使用mysql重新加载以重新创建转储的对象。 选项可用于修改SQL语句的格式，以及控制转储哪些对象。

* 有 --tab

> mysqldump为每个转储表生成两个输出文件。 服务器将一个文件写为制表符分隔的文本，每个表行一行。 此文件在输出目录中名为tbl_name.txt。 服务器还将表的CREATE TABLE语句发送到mysqldump，mysqldump将其写为输出目录中名为tbl_name.sql的文件。


## 参数

* -A, --all-databases 备份所有数据库
* -B, --databases 转储多个数据库，后跟数据库名称（可以多个）
* -d, --no-data 只导出表结构
* -t, --no-create-info 只导出数据
* -T, --tab=name 导出时表结构和数据分离（不建议用此参数），创建 .sql 和 .txt 文件 `注意：如果mysqldump与mysqld在同一台计算机上运行服务器才起作用`
* -F, --flush-logs
* --master-data[=#]
    * --master-data=1 生成的备份语句里添加 change master 语句及 binlog文件和位置点信息。
    * --master-data=2 生成的change master语句会被注释
* --single-transaction 适用于InnoDB数据库，用来保证备份的一致性，工作原理:设定本次会话隔离级别为REPEATABLE READ，以确保本次会话（dump）时，不会看到其他会话已经提交了的数据。
* -R, --routines 导出`存储过程`、`函数`
* -E, --events 导出event事件
* --triggers 导出每个转储表的触发器（默认为 `on` 使用 `--skip-triggers` 禁用）
* -x, --lock-all-tables 锁定所有数据库中的所有表。
* -l, --lock-tables (默认为 on; 使用 --skip-lock-tables 禁用) `关于表锁查看：show open tables where In_use>0;`

* -q, --quick 不缓冲查询（因为如果结果集很大的话会出问题），直接转储到stdout（默认为 on; 使用`--skip-quick to disable`禁用）
* --opt 等同 `--add-drop-table, --add-locks, --create-options, --quick, --extended-insert, --lock-tables, --set-charset, --disable-keys` 默认启用, 禁用使用`--skip-opt`

* -F, --flush-logs 导出之前刷新服务器中的bin-log日志文件 `可以确定全备和增量的临界点，生成新日志文件，未来增量恢复从这个日志文件开始`

> 接 `-F` ，注意：如果您一次转储多个数据库（使用选项--databases =或--all-databases），则将为每个转储的数据库刷新日志。

> 例外情况是使用--lock-all-tables或--master-data：在这种情况下，日志将只刷新一次，对应于所有表被锁定的时刻。 因此，如果您希望转储和日志刷新在同一时刻发生，则应使用--lock-all-tables或--master-data with --flush-logs。



## 实践

查看创建的数据库

```sql
mysql -uroot -p -S /data/mysql_3306/tmp/mysql.sock -e 'show databases' | egrep -vi 'Database|mysql|_schema|sys'
```


### mysqldump不加任何参数

```bash
# mkdir /backup

# mysqldump -uroot -p -S /data/mysql_3306/tmp/mysql.sock -A > /backup/alldb_$(date +%F).sql
Enter password:
Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events.
```

同时另开个终端查看

```sql
mysql> show open tables where In_use>0;
+-----------+--------------+--------+-------------+
| Database  | Table        | In_use | Name_locked |
+-----------+--------------+--------+-------------+
| employees | dept_manager |      1 |           0 |
| employees | titles       |      1 |           0 |
| employees | employees    |      1 |           0 |
| employees | dept_emp     |      4 |           0 |
| employees | departments  |      1 |           0 |
| employees | salaries     |      1 |           0 |
+-----------+--------------+--------+-------------+
```

```bash
head -n 30 /backup/alldb_2019-07-21.sql|egrep -v "#|^\/\*|--|^$"
```

### 备份表

> 去掉-B参数：第一个库名后面跟的都是表名

```bash
# mysqldump -uroot -p -S /data/mysql_3306/tmp/mysql.sock --master-data=2 employees departments > /backup/departments_$(date +%F).sql
Enter password:
Warning: A partial dump from a server that has GTIDs will by default include the GTIDs of all transactions, even those that changed suppressed parts of the database. If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events.
```

```bash
# cat /backup/departments_2019-07-21.sql
-- MySQL dump 10.13  Distrib 5.7.26, for linux-glibc2.12 (x86_64)
--
-- Host: localhost    Database: employees
-- ------------------------------------------------------
-- Server version       5.7.26-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
SET @MYSQLDUMP_TEMP_LOG_BIN = @@SESSION.SQL_LOG_BIN;
SET @@SESSION.SQL_LOG_BIN= 0;

--
-- GTID state at the beginning of the backup
--

SET @@GLOBAL.GTID_PURGED='e715f389-a77f-11e9-930c-0800271a40e6:1-45602';

--
-- Position to start replication or point-in-time recovery from
--

-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-binlog.000020', MASTER_LOG_POS=66374840;

--
-- Table structure for table `departments`
--

DROP TABLE IF EXISTS `departments`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `departments` (
  `dept_no` char(4) NOT NULL,
  `dept_name` varchar(40) NOT NULL,
  PRIMARY KEY (`dept_no`),
  UNIQUE KEY `dept_name` (`dept_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `departments`
--

LOCK TABLES `departments` WRITE;
/*!40000 ALTER TABLE `departments` DISABLE KEYS */;
INSERT INTO `departments` VALUES ('d009','Customer Service'),('d005','Development'),('d002','Finance'),('d003','Human Resources'),('d001','Marketing'),('d004','Production'),('d006','Quality Management'),('d008','Research'),('d007','Sales');
/*!40000 ALTER TABLE `departments` ENABLE KEYS */;
UNLOCK TABLES;
SET @@SESSION.SQL_LOG_BIN = @MYSQLDUMP_TEMP_LOG_BIN;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2019-07-21 11:47:33
```


### gzip压缩
```
# mysqldump -uroot -pDL2016-1q2w3e@_@db.com -S /data/mysql_3306/mysql.sock -B db1 | gzip > ~/db_$(date +%F).sql.gz
# ll -h ~/ |grep db
-rw-r--r--  1 root root 6.6M Jul  9 13:44 db_2016-07-09.sql
-rw-r--r--  1 root root 2.3M Jul  9 13:43 db_2016-07-09.sql.gz
```

**解压恢复**

* 方法1: gzip -d 压缩文件

```bash
# pwd
/root/backup
# ll
total 2332
-rw-r--r-- 1 root root 2376114 Jul  9 14:45 db1_2016-07-09.sql.gz
-rw-r--r-- 1 root root     793 Jul  9 14:57 db1.tb1_2016-07-09.sql.gz
-rw-r--r-- 1 root root     509 Jul  9 14:45 db2_2016-07-09.sql.gz

# gzip -d db1_2016-07-09.sql.gz 
[root@vb103 backup]# ll
total 6688
-rw-r--r-- 1 root root 6836848 Jul  9 14:45 db1_2016-07-09.sql
-rw-r--r-- 1 root root     793 Jul  9 14:57 db1.tb1_2016-07-09.sql.gz
-rw-r--r-- 1 root root     509 Jul  9 14:45 db2_2016-07-09.sql.gz
```
解压出来了，但压缩文件不存在了

不删除源压缩文件解压；

```
# gzip -cd db2_2016-07-09.sql.gz > db2_1.sql
# ll
total 6692
-rw-r--r-- 1 root root 6836848 Jul  9 14:45 db1_2016-07-09.sql
-rw-r--r-- 1 root root     793 Jul  9 14:57 db1.tb1_2016-07-09.sql.gz
-rw-r--r-- 1 root root    1382 Jul  9 16:56 db2_1.sql
-rw-r--r-- 1 root root     509 Jul  9 14:45 db2_2016-07-09.sql.gz
```

* 方法2: 

```bash
# gunzip < db2_2016-07-09.sql.gz | mysql -uroot -pDL2016-1q2w3e@_@db.com -S /data/mysql_3306/mysql.sock
```


### mysqldump 生产场景不同引擎备份命令

* MyISAM 引擎（适合所有引擎或混合引擎）

```
mysqldump -uroot -p -A -B -F -R --master-data=2 -x --events | gzip > /opt/all.sql.gz
```
> 提示：-F也可以取消，与--master-data有重复


* InnoDB 引擎

```
mysqldump -uroot -p -A -B -F -R --master-data=2 --events --single-transaction | gzip > /opt/all.sql.gz
```
> 提示：-F也可以取消，与--master-data有重复
