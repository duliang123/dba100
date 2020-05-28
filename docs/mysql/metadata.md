# 统计类SQL语句

## 表数量

查询指定数据库表的数量

```sql
SELECT COUNT(*) TABLES, table_schema FROM information_schema.TABLES WHERE table_schema = '数据库名称';
```

## 字段数量

```sql
-- 查询一个表中有多少字段
SELECT COUNT(*) FROM information_schema. COLUMNS WHERE table_schema = '数据库名称' AND table_name = '表名称';

-- 查询一个数据库中有多少字段
SELECT COUNT(column_name) FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = '数据库名称';
```

## 存储空间

统计表库存储空间大小

```sql
select table_schema,table_name,version,engine,data_length,table_rows from information_schema.tables where table_name='tb1';


SELECT TABLE_NAME, (DATA_LENGTH/1024/1024) as DataM ,(INDEX_LENGTH/1024/1024) as IndexM,((DATA_LENGTH+INDEX_LENGTH)/1024/1024) as AllM,TABLE_ROWS FROM information_schema.TABLES WHERE TABLE_SCHEMA = '数据库名称'
```

统计数据库存储空间（GB|MB）

```sql
-- 统计：数据占用空间大小
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from information_schema.TABLES where table_schema='数据库名称' 
-- 统计：数据+索引占用空间大小
select concat(round(sum(data_length/1024/1024),2)+round(sum(index_length/1024/1024),2),'MB') as data from information_schema.tables where table_schema='数据库名称';
```

## 数据量

查询每张表的数据量（多少条）

```sql
select table_name,table_rows from information_schema.tables where TABLE_SCHEMA = '数据库名称' order by table_rows desc;
```

统计数据库的总数据量（多少条）

```sql
SELECT sum(table_rows) from information_schema.tables where TABLE_SCHEMA = '数据库名称' order by table_rows desc;
```
