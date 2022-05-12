---
title: mysql日常命令行
date: 2021-06-16 10:23:20
tags:
   - mysql
---


1. 链接远程数据库

```js
mysql -h主机地址 -P端口 -u用户名 -p用户密码
mysql -h127.0.0.1 -P1234 -u"myname" -p"test123"
```



2. 显示所有库

```js
show databases;
```



3. 进入数据库

```js
use database_name;
```



4. 显示当前库中所有的表

```js
show tables;
```



5. 显示表字段

```js
desc table_name;
```



6. 创建表

```js
create table table_name (name char(100), path char(100), count int(10), firstName char(100), firstMD5 char(100), secondName char(100), secondMD5 char(100), thirdName char(100), thirdMD5 char(100));
```



7. 修改表名

```js
rename table table_name_old to table_name_new
```



8. 插入数据

```js
insert into table_name (name, path, count, firstName, firstMD5, secondName, secondMD5, thirdName, thirdMD5) VALUES ('test', 'test', 1, 'name1', 'md1', 'name2', 'md2', 'name3', 'md3');
```



9. 查询表中数据

```mysql
  select * from table_name;

  select * from table_name where name = 'test';

  select * from table_name order by id limit 0,2;
```



10. 更新表中某一行数据

```mysql
update table_name set folderName ='Mary' where id=1;
```



11. 删除表中某一行数据

```mysql
delete from table_name where folderName = 'test';
```



12. 添加表字段

```mysql
alter table table_name add id int auto_increment not null primary key;
```



13. 删除字段

```mysql
alter table table_name drop folderName;
```