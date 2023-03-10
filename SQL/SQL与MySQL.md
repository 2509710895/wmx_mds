# SQL 与 MySQL

Structure Query Language(结构化查询语言)简称 SQL。数据库是长期存放在计算机内,有组织,可共享的大量数据的集合,是一个数据 "仓库"，是云端服务以及边缘设备进行数据持久化存储的基础工具。关系型数据库是通过外键关联来建立表与表之间的关系的数据库类型。MySQL 即是关系型数据库的一种。

## （一）数据库的基本操作

1.创建数据库

```
create database 'database_name';
```

2.查看数据库

```
show databases;`or `show database 'database_name';
```

3.删除数据库

```
drop database 'database_name';
```

4.切换数据库

```
use 'database_name';
```

## （二）数据表的基本操作

1.创建数据表

```
create table 'table_name';
```

2.查看数据表

```
show tables/table 'table_name';`or `desc 'table_name';
```

3.修改数据表

修改表名

```
alter table 'old_name' rename to 'new_name';
```

修改字段名

```
alter table 'table_name' change 'old_name' 'new_name' varchar(10);
```

修改字段数据类型

```
alter table 'table_name' modify 'old_name' 'new_type';
```

增加字段

```
alter table 'table_name' add 'new_name' varchar(50);
```

删除字段

```
alter table 'table_name' drop 'old_name';
```

## （三）数据的基本操作

1.增

```
INSERT INTO 表名（字段名1,字段名2,...) VALUES (值 1,值 2,...), (值 1,值 2,...);
```

2.删

```
DELETE FROM 表名 [WHERE 条件表达式];
```

3.改

```
UPDATE 表名 SET 字段名1=值1[,字段名2 =值2,…] [WHERE 条件表达式];
```

4.查

```
select *(字段名1,字段名2,...) from student [WHERE 条件表达式];
```
