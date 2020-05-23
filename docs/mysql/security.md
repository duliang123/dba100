# 安装之后安全设置 

安装完`MySQL`之后，首先需要修改密码。

```sql
shell> mysql -uroot -p

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```
