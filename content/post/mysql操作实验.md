---
title: "Mysql操作实验"
date: 2020-04-04T22:26:40+08:00
draft: false
categories: ["数据库"]
tags: ["mysql"]
---



## 实验1.1 数据库定义语言实验

> （1）实验目的 
>
> 理解和掌握数据库DDL语言，能够熟练地使用SQL DDL语句创建、修改和删除数据库、模式和基本表。 
>
> （2）实验内容和要求 
>
> 理解和掌握SQL DDL语句的语法，特别是各种参数的具体含义和使用方法；使用SQL语句创建、修改和删除数据库、模式和基本表。掌握SQL语句常见语法错误的调试方法。 
>
> （3）实验重点和难点 
>
> 实验重点：创建数据库、基本表。 实验难点：创建基本表时，为不同的列选择合适的数据类型，正确创建表级和列级完整性约束，如列值是否允许为空、主码和外码等。注意：数据完整性约束，可以在创建基本表时定义，也可以先创建表然后定义完整性约束。由于完整性约束的限制，被引用的表要先创建。



#### 创建一个 sales 数据库

```sql
create database sales character set utf8;
```

#### 创建一个region表

以 id 为主码

```sql
use sales;

create table region(
    id int primary key,
    name varchar(255),
    alias varchar(255)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 从csv文件导入数据

```sql
load data infile '/mnt/e/课程笔记/数据库/实验课安排2020/dbtestdata_csv/region.csv'
into table region character set utf8
fields terminated by ',' optionally enclosed by '"' escaped by '"'
lines terminated by '\r\n';
```

但提示

```
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

查阅资料后, 是用下面的方法解决

找到mysql配置文件 `/etc/mysql/mysql.conf.d/mysqld.cnf` , 在`[mysqld]`下, 设置

```
secure_file_priv = ''
```

即可成功导入数据

导入数据之后, 执行select查看数据

```
mysql> select * from region;
+------+---------------------------------+-----------------+
| id   | name                            | alias           |
+------+---------------------------------+-----------------+
|    1 | 亚洲                            | 新兴的大陆      |
|    2 | 非洲                            | 战乱的大陆      |
|    3 | 北美洲                          | 美国的地盘      |
|    4 | 南美洲                          | 美国的后院      |
|    5 | 欧洲                            | 美国的帮手      |
|    6 | 大洋洲                          | 小国的地盘      |
+------+---------------------------------+-----------------+
6 rows in set (0.00 sec)
```



#### 删除数据库

```
drop database test;
```



#### 删除一个表

```
drop table test;
```



#### 清空表数据

```
delete from table_name;
```



### 一系列的操作

注释解释了这个操作的含义

```mysql
# 使用数据库 sales
use sales;

# 设置主键
alter table region add primary key(id);

# 取消主键
alter table region drop primary key;

# 创建一个有主键的table
create table table2 (
    table2_id int not null,
    primary key (table2_id)
);

# 删除表
drop table table2;

# 建表时添加外键. 注意这里table2的这个列需要是主键
create table table1(
    table1_id int not null,
    table2_id int not null,
    CONSTRAINT `FK1` foreign key(table2_id) references table2(table2_id)
);

# 删除外键约束. 注意要对应外键约束的名字
alter table sales.table1 drop foreign key table1_ibfk_1;

# 单独设置外键约束（注意这里是外键约束而不是外键，外键是一列，而外键约束是一个约束）
# 不使用 constraint `FK_hello` 做别名会分配一个名字
alter table table1 add constraint `FK_hello` foreign key FK_name(table2_id) references table2(table2_id);

# 设置唯一键
create table tableName(
    columnName int,
    unique key(columnName)
);

# 单独添加唯一键
alter table tableName add unique key(columnName)

# 删除唯一键
alter table tableName drop index columnName;
```



## 实验1.2 数据基本查询实验

> （1）实验目的 
>
> 掌握SQL程序设计基本规范，熟练运用SQL语言实现数据基本查询，包括单表查询、分组统计查询和连接查询。 
>
> （2）实验内容和要求 
>
> 针对某个数据库设计各种单表查询SQL语句、分组统计查询语句；设计单个表针对自身的连接查询，设计多个表的连接查询。理解和掌握SQL查询语句各个子句的特点和作用，按照SQL程序设计规范写出具体的SQL查询语句，并调试通过。
>
> 说明：简单地说，SQL程序设计规范包含SQL关键字大写、表名、属性名、存储过程名等标识符大小写混合、SQL程序书写缩进排列等编程规范。 
>
> （3）实验重点和难点 
>
> 实验重点：分组统计查询、单表自身连接查询、多表连接查询。 实验难点：区分元组过滤条件和分组过滤条件；确定连接属性，正确设计连接条件。



### 创建两个表

首先创建两个表，分别为region，nation

#### 创建 region

```mysql
# 创建表
create table region(
    id int primary key,
    name varchar(255),
    alias varchar(255)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

从csv文件中导入数据

```mysql
# 从csv文件中导入数据
load data infile '/mnt/e/课程笔记/数据库/实验课安排2020/dbtestdata_csv/region.csv'
into table sales.region character set utf8
fields terminated by ',' optionally enclosed by '"' escaped by '"'
lines terminated by '\r\n';
```



#### 创建 nation

并设置 nation 的键 region_id 作为 region.id 的外键

```mysql
# 建立 nation 国家表
create table nation(
    id int primary key,
    cn_name varchar(255),
    region_id int,
    en_name varchar(255),
    CONSTRAINT `fk_region_id` foreign key(region_id) references region(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

导入数据

```mysql
# 导入数据
load data infile '/mnt/e/课程笔记/数据库/实验课安排2020/dbtestdata_csv/nation.csv'
into table nation character set utf8
fields terminated by ',' optionally enclosed by '"' escaped by '"'
lines terminated by '\r\n';
```



### 查看表信息

下表是 `region` 的信息（代表6个大洲）。 其中id为大洲的id， name为大洲的中文名字， alias为大洲的别名

```mysql
mysql> describe region;
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| id    | int(11)      | NO   | PRI | NULL    |       |
| name  | varchar(255) | YES  |     | NULL    |       |
| alias | varchar(255) | YES  |     | NULL    |       |
+-------+--------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

mysql>
```

查看所有列

```
mysql> select * from region;
+----+---------------------------------+-----------------+
| id | name                            | alias           |
+----+---------------------------------+-----------------+
|  1 | 亚洲                            | 新兴的大陆      |
|  2 | 非洲                            | 战乱的大陆      |
|  3 | 北美洲                          | 美国的地盘      |
|  4 | 南美洲                          | 美国的后院      |
|  5 | 欧洲                            | 美国的帮手      |
|  6 | 大洋洲                          | 小国的地盘      |
+----+---------------------------------+-----------------+
6 rows in set (0.00 sec)

mysql>
```



下表是 nation 的信息。

id 代表国家的id，cn_name代表国家的中文名, region_id代表所在的大洲的id（外键约束）, en_name代表国家的英文名.

```mysql
mysql> describe nation;
+-----------+--------------+------+-----+---------+-------+
| Field     | Type         | Null | Key | Default | Extra |
+-----------+--------------+------+-----+---------+-------+
| id        | int(11)      | NO   | PRI | NULL    |       |
| cn_name   | varchar(255) | YES  |     | NULL    |       |
| region_id | int(11)      | YES  | MUL | NULL    |       |
| en_name   | varchar(255) | YES  |     | NULL    |       |
+-----------+--------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

mysql>
```

展示部分列

```
mysql> select * from nation;
+-----+----------------------+-----------+------------------------------+
| id  | cn_name              | region_id | en_name                      |
+-----+----------------------+-----------+------------------------------+
|   1 | 阿富汗                |         1 | Afghanistan                  |
|   2 | 阿尔巴尼亚           |         1 | Albania                      |
|   3 | 安道尔               |         1 | Andorra                      |
|   4 | 安哥拉               |         2 | Angola                       |
|   8 | 阿根廷               |         4 | Argentina                    |
|  11 | 澳大利亚             |         6 | Australia                    |
|  12 | 奥地利               |         5 | Austria                      |
|  13 | 阿塞拜疆             |         1 | Azerbaijan                   |
|  14 | 阿联酋               |         1 | United Arab Emirates         |
|  15 | 巴哈马               |         3 | Bahamas                      |
|  16 | 巴林                 |         1 | Bahrain                      |
|  17 | 孟加拉               |         1 | Bangladesh                   |
|  21 | 比利时               |         5 | Belgium                      |
|  22 | 贝宁                 |         2 | Benin                        |
|  23 | 百慕大               |         3 | Bermuda                      |
|  24 | 不丹                 |         1 | Bhutan                       |
|  25 | 玻利维亚             |         4 | Bolivia                      |
|  27 | 博茨瓦纳             |         2 | Botswana                     |
|  29 | 巴西                 |         4 | Brazil                       |
|  30 | 文莱                 |         1 | Brunei Darussalam            |
|  31 | 保加利亚             |         5 | Bulgaria                     |
|  33 | 布隆迪               |         2 | Burundi                      |
|  35 | 喀麦隆               |         2 | Cameroon                     |
|  36 | 加拿大               |         3 | Canada                       |
|  37 | 佛得角               |         2 | Cape Verde Republic of       |
|  38 | 中非共和国           |         2 | The Central African Republic |
```



#### 查询所有数据

```
select * from region;
```

查询`id < 3`的数据

```mysql
select * from region
where id < 3;
```

得到

```
mysql> select * from region
    -> where id < 3;
+----+-------------------------------+-----------------+
| id | name                          | alias           |
+----+-------------------------------+-----------------+
|  1 | 亚洲                          | 新兴的大陆      |
|  2 | 非洲                          | 战乱的大陆      |
+----+-------------------------------+-----------------+
2 rows in set (0.00 sec)

mysql>
```



#### 查询在北美洲的所有国家 

```mysql
select id, cn_name from nation
where region_id = 3;
```

结果

```
mysql> select id, cn_name from nation
    -> where region_id = 3;
+-----+-------------------------------------+
| id  | cn_name                             |
+-----+-------------------------------------+
|  15 | 巴哈马                              |
|  23 | 百慕大                              |
|  36 | 加拿大                              |
|  47 | 哥斯达黎加                          |
|  49 | 古巴                                |
|  79 | 格林纳达                            |
|  82 | 危地马拉                            |
|  87 | 海地                                |
|  88 | 洪都拉斯                            |
| 100 | 牙买加                              |
| 135 | 墨西哥                              |
| 151 | 尼加拉瓜                            |
| 161 | 巴拿马                              |
| 212 | 美国                                |
| 217 | 委内瑞拉                            |
+-----+-------------------------------------+
15 rows in set (0.00 sec)

mysql>
```



#### 查询所有每个洲的国家的数量

```
select region_id as id, count(*) cnt from nation
group by region_id;
```

结果

```
mysql> select region_id as id, count(*) cnt from nation
    -> group by region_id;
+------+-----+
| id   | cnt |
+------+-----+
|    1 |  46 |
|    2 |  37 |
|    3 |  15 |
|    4 |  10 |
|    5 |  35 |
|    6 |   7 |
+------+-----+
6 rows in set (0.00 sec)

mysql>
```





## 实验1.3 数据高级查询实验

> （1）实验目的 
>
> 掌握SQL嵌套查询和集合查询等各种高级查询的设计方法等。 
>
> （2）实验内容和要求 
>
> 针对自定义数据库，正确分析用户查询要求，设计各种嵌套查询和集合查询。 
>
> （3）实验重点和难点 
>
> 实验重点：嵌套查询。 
>
> 实验难点：相关子查询、多层EXIST嵌套查询。



#### 查询每个大洲各有多少个国家

```mysql
select 
	region.id as region_id, region.name as region_name, region_count.cnt as count
from region, (
	select region_id as id, count(*) cnt from nation
	group by region_id
) region_count
where 
	region.id = region_count.id;
```

方法是用一个包含group的select对nation操作建立一个临时表(统计每个region_id出现的次数), 然后和region组合查询. 

```
mysql> select 
    -> region.id as region_id, region.name as region_name, region_count.cnt as count
    -> from region, (
    -> select region_id as id, count(*) cnt from nation
    -> group by region_id
    -> ) region_count
    -> where
    -> region.id = region_count.id;
+-----------+---------------------------------+-------+
| region_id | region_name                     | count |
+-----------+---------------------------------+-------+
|         1 | 亚洲                            |    46 |
|         2 | 非洲                            |    37 |
|         3 | 北美洲                          |    15 |
|         4 | 南美洲                          |    10 |
|         5 | 欧洲                            |    35 |
|         6 | 大洋洲                          |     7 |
+-----------+---------------------------------+-------+
6 rows in set (0.00 sec)

mysql>
```



#### 查询以A或B开头的行, 并且按照字典序递增

```mysql
select cn_name, en_name from nation
where cn_name not in (
	select cn_name from nation 
    where en_name not like 'A%' and en_name not like 'B%'
)
order by en_name;
```

结果

```
mysql> select cn_name, en_name from nation
    -> where cn_name not in (
    -> select cn_name from nation
    ->     where en_name not like 'A%' and en_name not like 'B%'
    -> )
    -> order by en_name;
+-------------------------------------+-------------------+
| cn_name                             | en_name           |
+-------------------------------------+-------------------+
| 阿富汗                              | Afghanistan       |
| 阿尔巴尼亚                          | Albania           |
| 阿尔及利亚                          | Algeria           |
| 美国                                | America           |
| 安道尔                              | Andorra           |
| 安哥拉                              | Angola            |
| 阿根廷                              | Argentina         |
| 澳大利亚                            | Australia         |
| 奥地利                              | Austria           |
| 阿塞拜疆                            | Azerbaijan        |
| 巴哈马                              | Bahamas           |
| 巴林                                | Bahrain           |
| 孟加拉                              | Bangladesh        |
| 比利时                              | Belgium           |
| 贝宁                                | Benin             |
| 百慕大                              | Bermuda           |
| 不丹                                | Bhutan            |
| 玻利维亚                            | Bolivia           |
| 博茨瓦纳                            | Botswana          |
| 巴西                                | Brazil            |
| 文莱                                | Brunei Darussalam |
| 保加利亚                            | Bulgaria          |
| 布隆迪                              | Burundi           |
+-------------------------------------+-------------------+
23 rows in set (0.00 sec)

mysql> 
```





## 实验1.4 数据更新实验

> （1）实验目的 
>
> 熟悉数据库的数据更新操作，能够使用SQL语句对数据库进行数据的插入、修改、删除操作。 
>
> （2）实验内容和要求 
>
> 针对自定义数据库设计单元组插入、批量数据插入、修改数据和删除数据等SQL语句。理解和掌握INSERT、UPDATE和DELETE语法结构的各个组成成分，结合嵌套SQL子查询，分别设计几种不同形式的插入、修改和删除数据的语句，并调试成功。 
>
> （3）实验重点和难点
>
> 实验重点：插入、修改和删除数据的SQL。
>
> 实验难点：与嵌套SQL子查询相结合的插入、修改和删除数据的SQL语句；利用一个表的数据来插入、修改和删除另外一个表的数据。

创建新表来进行这部分测试

```mysql
create table student (
    id int primary key,
    name varchar(255),
    age int
);

create table student2 (
    id int primary key,
    age int
);
```



### 插入

#### 插入一行或多行

```mysql
insert into student (id, name, age)
values
(1, "小明", 18),
(2, "小红", 81),
(3, "老王", 36);
```

可以指定对应的列名(或者不指定, 按默认). 可以多行插入

结果

```
mysql> insert into student (id, name, age)
    -> values
    -> (1, "小明", 18),
    -> (2, "小红", 81),
    -> (3, "老王", 36);
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select * from student;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   18 |
|  2 | 小红   |   81 |
|  3 | 老王   |   36 |
+----+--------+------+
3 rows in set (0.00 sec)

mysql>
```



#### 从select语句中插入

```
insert into student2 
(id, age)
select id, age from student
where age < 60;
```

结果

```mysql
mysql> insert into student2 
    -> (id, age)
    -> select id, age from student
    -> where age < 60;
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from student2;
+----+------+
| id | age  |
+----+------+
|  1 |   18 |
|  3 |   36 |
+----+------+
2 rows in set (0.00 sec)

mysql>
```



### 更新

将id = 1的学生年龄增加40岁

```
update student set age=age+40
where id=1;
```

结果

```mysql
mysql> update student set age=age+40
    -> where id=1;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from student;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   58 |
|  2 | 小红   |   81 |
|  3 | 老王   |   36 |
+----+--------+------+
3 rows in set (0.00 sec)

mysql>
```



将小明小红中的小都修改为大, 将老改为old.

```
update student set 
name=replace(name, "小", "大"),
name=replace(name, "老", "old");
```

结果

```mysql
mysql> select * from student;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 大明   |   58 |
|  2 | 大红   |   81 |
|  3 | old王  |   36 |
+----+--------+------+
3 rows in set (0.00 sec)

mysql>
```



### 删除

#### 删除表中全部的数据

```
delete from student;
```

结果

```
mysql> delete from student;
Query OK, 3 rows affected (0.02 sec)

mysql> select * from student;
Empty set (0.00 sec)

mysql>
```



#### 删除部分数据

```
delete from student
where name not like "小%";
```

结果

```
mysql> insert into student (id, name, age) values (1, "小明", 18), (2, "小红", 81), (3, "老王", 36);
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select * from student;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   18 |
|  2 | 小红   |   81 |
|  3 | 老王   |   36 |
+----+--------+------+
3 rows in set (0.00 sec)

mysql> delete from student where name not like "小%";
Query OK, 1 row affected (0.02 sec)

mysql> select * from student;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   18 |
|  2 | 小红   |   81 |
+----+--------+------+
2 rows in set (0.00 sec)

mysql>
```



## 实验1.5 视图

> （1）实验目的 
>
> 熟悉SQL语言有关视图的操作，能够熟练使用SQL语句来创建需要的视图，定义数据库外模式，并能使用所创建的视图实现数据管理。 
>
> （2）实验内容和要求 
>
> 针对给定的数据库模式，以及相应的应用需求，创建视图和带WITH CHECK OPTION的视图，并验证视图WITH CHECK OPTION选项的有效性。理解和掌握视图消解执行原理，掌握可更新视图和不可更新视图的区别。 
>
> （3）实验重点和难点
>
> 实验重点：创建视图。 
>
> 实验难点：可更新的视图和不可更新的视图的区别， WITH CHECK OPTION的验证。



### 创建视图

创建视图, 并查看所有的表

```
create view stview as select * from student;
```

结果

```
mysql> create view stview as select * from student;
Query OK, 0 rows affected (0.02 sec)

mysql> show tables;
+-----------------+
| Tables_in_sales |
+-----------------+
| nation          |
| region          |
| student         |
| student2        |
| stview          |
| tableName       |
| tableName1      |
| tableName2      |
+-----------------+
8 rows in set (0.00 sec)

mysql>
```

可以看到视图"stview"在tables中显示



使用

```sql
drop view stview;
```

撤销视图



### 更新视图

将视图中的数据更新, 再查看原表中的数据, 发现同时也更新了.

```
mysql> update stview set age=20 where id=1;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from stview;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   20 |
|  2 | 小红   |   81 |
+----+--------+------+
2 rows in set (0.00 sec)

mysql> select * from student;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   20 |
|  2 | 小红   |   81 |
+----+--------+------+
2 rows in set (0.00 sec)

mysql>
```

将原表更新, 视图也随之更新

```
mysql> select * from student;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |  120 |
|  2 | 小红   |  181 |
+----+--------+------+
2 rows in set (0.00 sec)

mysql> select * from stview;
+----+--------+
| id | name   |
+----+--------+
|  1 | 小明   |
|  2 | 小红   |
+----+--------+
2 rows in set (0.00 sec)

mysql> update student set name=replace(name, '小', '老');
Query OK, 2 rows affected (0.02 sec)
Rows matched: 2  Changed: 2  Warnings: 0

mysql> select * from stview;
+----+--------+
| id | name   |
+----+--------+
|  1 | 老明   |
|  2 | 老红   |
+----+--------+
2 rows in set (0.00 sec)
```



### 是否可更新

简单视图是可更新的。可以在UPDATE、DELETE或INSERT等语句中使用它们，以更新基表的内容。对于可更新的视图，在视图中的行和基表中的行之间必须具有一对一的关系。

如果视图包含下述结构中的任何一种，那么它就是不可更新的：

1. 聚合函数（SUM(), MIN(), MAX(), COUNT()等）。
2. DISTINCT
3. GROUP BY
4. HAVING
5. UNION或UNION ALL
6. 位于选择列表中的子查询
7. Join
8. FROM子句中的不可更新视图
9. WHERE子句中的子查询，引用FROM子句中的表。
10. 仅引用文字值（在该情况下，没有要更新的基本表）。
11. ALGORITHM = TEMPTABLE（使用临时表总会使视图成为不可更新的）。



### WITH CHECK OPTION

WITH CHECK OPTION 是为了保证视图的一致性

>  可以为可更新视图指定 WITH CHECK OPTION 子句，以防止插入 select_statement 中 WHERE 子句不为真的行。它还会阻止更新 WHERE 子句为true的行，但更新会导致它不为true（换句话说，它会阻止可见行更新为不可见的行）

比如创建如下表

```mysql
create table test (
    id int primary key,
    name varchar(255),
    age int
);
```

插入以下数据

```mysql
insert into test (id, name, age)
values
(1, "小明", 18),
(2, "小红", 81),
(3, "老王", 36),
(4, "老张", 66),
(5, "老李", 54),
(6, "小孙", 333);
```



#### 建立没有WITH CHECK OPTION 的视图

```
create view view_test1
as 
select * from test
where name like "小%";
```

查看这个视图

```
mysql> select * from view_test1;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   18 |
|  2 | 小红   |   81 |
|  6 | 小孙   |  333 |
+----+--------+------+
3 rows in set (0.00 sec)

mysql>
```

然后向其中插入name为"老李"的行

```
insert into view_test1 values
(7, "老李", 666);
```

看到插入成功, 视图中没有(因为不满足where条件), 原表中有

```
mysql> insert into view_test1 values
    -> (7, "老李", 666);
Query OK, 1 row affected (0.01 sec)

mysql> select * from view_test1;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   18 |
|  2 | 小红   |   81 |
|  6 | 小孙   |  333 |
+----+--------+------+
3 rows in set (0.01 sec)

mysql> select * from test;
+----+--------+------+
| id | name   | age  |
+----+--------+------+
|  1 | 小明   |   18 |
|  2 | 小红   |   81 |
|  3 | 老王   |   36 |
|  4 | 老张   |   66 |
|  5 | 老李   |   54 |
|  6 | 小孙   |  333 |
|  7 | 老李   |  666 |
+----+--------+------+
7 rows in set (0.00 sec)

mysql>
```



#### 建立有 WITH CHECK OPTION  的视图

```mysql
create view view_test2
as 
select * from test
where name like "小%"
WITH CHECK OPTION;
```

然后向其中插入name为“老李”的行

```
insert into view_test2 values
(8, "老李", 666);
```

结果插入失败

```
mysql> insert into view_test2 values
    -> (8, "老李", 666);
ERROR 1369 (HY000): CHECK OPTION failed 'sales.view_test2'
mysql>
```



试图将视图中的"小明"改成"李明"

```
update view_test2 set name=replace(name, "小明", "李明");
```

得到错误

```
mysql> update view_test2 set name=replace(name, "小明", "李明");
ERROR 1369 (HY000): CHECK OPTION failed 'sales.view_test2'
mysql>
```

改成"小李明", 成功

```
mysql> update view_test2 set name=replace(name, "小明", "小李明");
Query OK, 1 row affected (0.02 sec)
Rows matched: 3  Changed: 1  Warnings: 0

mysql>
```