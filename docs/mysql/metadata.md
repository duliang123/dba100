# 运维统计类元数据SQL

统计表大小

```sql
use information_schema;
select table_schema,table_name,version,engine,data_length,table_rows from tables where table_name='tb1';
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from TABLES where table_schema='db1' 
```

