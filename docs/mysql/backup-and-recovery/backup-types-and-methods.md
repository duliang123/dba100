# 备份恢复类型与方法 

## 备份类型

* 物理（原始）与逻辑备份
* 在线与离线备份
* 本地与远程备份
* 快照备份 `Veritas` `LVM` `ZFS`
* 完全与增量备份
* 完全与时间点（增量）恢复

## 备份方法

* 使用`MySQL Enterprise Backup`进行热备份
* 使用`mysqldump`进行逻辑备份
* 通过复制表文件`Copying Table Files`进行备份 `关停MySQL服务器或 FLUSH TABLES tbl_list WITH READ LOCK;后操作`
* 制作分隔文本文件备份 `SELECT * INTO OUTFILE 'file_name' FROM tbl_name`；要重新加载分隔文本数据文件使用 `LOAD DATA`语句
* 通过启用二进制日志进行增量备份
* 使用从库复制进行备份
* 使用LVM快照备份

