---
layout: article
tags: MySQL sql
title: MySQL学习笔记
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

MySQL数据库的安装、配置以及用法

<!--more-->

# 安装

## 树莓派上

- 安装：`sudo apt-get install mysql-server`
- 卸载：`sudo apt-ge autoremove --purge mysql-server`

## centos7

- 首先创建一个配置文件
```
# sudo vim /etc/yum.repos.d/MariaDB.repo
```

官方针对不同的系统给出了不同的配置文件内容，具体可以在[官网](https://downloads.mariadb.org/mariadb/repositories/)查看，比如centos7：
```
# MariaDB 10.4 CentOS repository list - created 2019-09-09 06:32 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

- 然后可以安装：
```
# sudo yum install MariaDB-server MariaDB-client -y
# sudo systemctl start mariadb.service       // 启动mysql
# sudo systemctl enable mariadb.service      // 设置开机自动启动
```

更换源之后如果提示安装包不存在，可以先执行`sudo yum clean all`再进行安装

之后进行安全性的配置：
```
# sudo mysql_secure_installation
```

输入命令之后根据提示进行配置就可以了

- 开放端口

centos由于有防火墙的存在，默认是无法远程连接mysql的，需要在firewalld中为mysql开放端口

首先查看当前firewalld的zones：
```
# sudo firewall-cmd --get-active-zones
```

结果中一般只有一个public，如果有其他的，需要为每一个zone都开放端口（将下面的public替换成对应的zone名称）：
```
# sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
# sudo firewall-cmd --reload         // 重启防火墙
```

- 卸载mysql

首先查看安装了哪些包：
```
# yum list installed mariadb*
```

把他们全部卸载掉：
```
# yum remove pkg_name
```

之后还要删除一些残留的文件

先查看这些文件是否存在：
```
# ls /etc/my.cnf
# ll /var/lib/mysql/
```

删除：
```
# rm -rf /etc/my.cnf
# rm -rf /var/lib/mysql/
```

# 配置

- 设置密码
树莓派上安装mysql默认是没有密码的，需要自己添加
  - 首先空密码登陆`sudo mysql -u root`
  - 设置root账户密码
  ```
  use mysql;
  update user set plugin='mysql_native_password' where user='root';
  UPDATE user SET password=PASSWORD('你自己的密码') WHERE user='root';
  flush privileges;
  exit;
  ```
  - 注意每一条mysql的命令都需要以`;`为结尾
  - 设置成功后就可以通过密码登陆`sudo mysql -h localhost -u root -p`，-h表示host，-u表示user，后面要带参数，-p表示password密码登陆

## 创建新用户并授权

### 1.创建用户
```
create user 'username'@'host' identified by 'password'
```
- 其中`username`表示新建用户的用户名
- `host`表示可以在那些主机可以登录，可以指定主机的ip，如果要设置所有主机可登录，将`host`替换为`%`
- `password`表示该用户登录的密码
  
### 2.授权

新建的用户是没有数据库的操作权限的，需要为其授权：

```
grant privileges on databasename.tablename to 'username'@'host'
```
- 其中`privileges`表示具体赋予哪一项权限，如`update` `delete`等，如果要赋予所有操作权限，将`privileges`替换成`all`
- `databasename`表示该权限是针对哪一个数据库，如果对于所有数据库都赋予该权限，将`databasename`替换成`*`
- `tablename`表示权限针对哪一个表格，如果对于所有表格都赋予该权限，将`tablename`替换成`*`
- 注意二者之间的`.`不要落下
- `username`和`host`意义与上面一样
  
### 3.撤销权限
```
revoke privileges on databasename.tablename to 'username'@'host'
```
各参数的意义与上面相同

### 4.删除用户
```
drop user 'username'@'host'
```

## 修改远程连接配置文件

- 对于树莓派
修改配置文件
```
# sudo vi /etc/mysql/mariadb.conf.d/50-server.cnf
```
注释掉其中的`bind-address  = 127.0.0.1`这一行  
然后保存，重启

## 忘记密码

这里针对MariaDB在Centos7系统上的情况，如果忘记root密码，可以首先修改配置文件`vim /etc/my.cnf`

在其中添加

```
[mysqld]
skip-grant-tables
```

这样就能在登录的时候跳过权限认证，然后重启MariaDB

```
# sudo systemctl restart mariadb
```

然后在命令行输入`mysql`进入mysql

首先刷新权限
```
MaiaDB [(none)]> FLUSH PRIVILEGES;
```

设置新密码

```
MaiaDB [(none)]> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

然后再刷新权限

```
MaiaDB [(none)]> FLUSH PRIVILEGES;
```

这样新的root密码就修改好了

# 常用命令

参考[官方文档](https://dev.mysql.com/doc/refman/8.0/en/)

## 1.新建

- 可以使用

```
show databases;
```
查看已有数据库

- 对于已有的数据库，使用`use`命令进入该数据库（此命令可不加分号），比如：

```
use mysql
```

- 若想创建一个新的数据库，使用`create`命令:
```
create database your_table
```

- 进入数据库之后，使用
```
show tables;
```
查看表格

- 同样可以使用`create`命令来创建表格

```
create table your_table (column1 varchar(10), column2 varchar(20), column3 char(1), column4 date);
```
其中，`your_name`为创建的表格名称，`column`为列标签，后面带的参数表示这一列的数据类型和长度

- 使用`describe`命令查看表格的内容：

```
describe your_table;
```

## 2.为数据库添加数据

可以从文件中加载数据添加到数据库表格中

- 从文件：首先编辑一个`pet.txt`文件，各项之间用`tab`隔开，没有的值使用`\N`或者`NULL`代替，如：

```
Whistler        Gwen    bird    \N      1997-12-09      \N
```
注意这里面的各项要对应创建表格时候的各个列

然后使用load命令加载：

```sql
LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet
```

- 如果需要为表格增加新行，使用`insert`命令

```sql
INSERT INTO pet VALUES ('Puffball','Diane','hamster','f','1999-03-30',NULL);
```
  
## 3.修改数据库中的数据

使用`update`命令

如：将name为Browser的宠物birth改为1989-08-31：

```sql
UPDATE pet SET birth = '1989-08-31' WHERE name = 'Bowser';
```

## 4.从数据库中提取数据

使用`select`命令，主要格式为：

```sql
SELECT 'what_to_select' // 选择哪一项数据，可以用 * 表示全部数据
FROM 'which_table' // 从哪一个表格
WHERE 'conditions_to_satisfy'; // 满足哪些条件
```

在`where`的条件判断中可以使用`and`和`or`等逻辑语句

- 选择数据并排序：
  - 使用`order`命令
  - 比如需要通过生日来排序，并只查看名字和生日两项:`SELECT name, birth FROM pet ORDER BY birth;`
  - 排序默认为升序，可以通过`desc`设置为降序：`SELECT name, birth FROM pet ORDER BY birth DESC;`
  
- `max()`函数寻找最大值

## 5.删除行

```sql
DELETE FROM tablename WHERE condition;
```

`tablename`为要操作的表，`condition`为删除的行所满足的条件

## 6.为数据分组

使用`group by`语句，将表中的行按照所选择的相同值来分组

## 7.LIKE查找

按`like 'b%' '%f' '%w%' `形式查找，也可以使用`_`下划线占位按长度查找

## 8.JOIN方法将两个表关联起来

- INNER JOIN（内连接,或等值连接）：获取两个表中字段匹配关系的记录。（NATURAL JOIN）
- LEFT JOIN（左连接）：获取左表所有记录，即使右表没有对应匹配的记录。
- RIGHT JOIN（右连接）： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

## 9.AS方法

`AS`可以将一个一个元素暂时命名为另一个元素，而且不修改原来的元素
如`...TABLE pet AS p1 ...;`
或者`SELECT name as n1 FROM pet;`
  
## 10.日期计算

使用`TIMESTAMPDIFF()`命令，需要传递三个参数：

- 1.要计算的日期参数，可以是YEAR,MONTH,DAY等
- 2.减数
- 3.被减数
- 比如：`SELECT name, birth, CURDATE(), TIMESTAMPDIFF(YEAR,birth,CURDATE()) AS age FROM pet;`
  - 其中`DURDATE()`是内置函数，返回当前日期
  - `AS`表示将其前面的内容（TIMESTAMPDIFF那一串）作为其后面的参数（age）显示在pet表格中
  
## 11.修改表中列的名称、类型

```sql
ALTER TABLE 'tablename'  MODIFY COLUMN 'column_name'  'new_name'  'new_type'    新默认值     新注释;  
ALTER TABLE   'table1'   MODIFY COLUMN   'column1'    'column2'  'float(4,2)'  DEFAULT NULL  COMMENT '注释';  // 示例
```

## 12.插入新列

```sql
ALTER TABLE 'tablename'
	ADD COLUMN 'colomnname' 'type' NOT NULL AFTER `columnname`;
```

在指定位置插入一列，`AFTER`后面指的是插入在那一列的后面

#### 插入自增列

插入名为`id`的自增列并将其设为主键（FIRST表示插入到第一列）

```sql
ALTER TABLE tablename ADD id INT AUTO_INCREMENT PRIMARY KEY FIRST
```


## 13.查询前几行或后几行

使用`MILIT`语句，LIMIT后可以接受一个或两个整数参数：

```sql
SELECT * FROM table LIMIT 5;            // 查询前5行
SELECT * FROM table LIMIT 5, 10;        // 查询从第6行开始的10行（6-15）
SELECT * FROM table LIMIT 5, -1;        // 查询第6行到最后一行
```

可以通过加上`DESC`倒叙排列来查询后几行：

```sql
SELECT * FROM table WHERE id>$id ORDER BY id DESC LIMIT 5;     // 查询最后5行
```

## 14.查看当前运行的命令

```
show full processlist;
```

# 常见问题

## 1.中文字符

有时为了在数据库中存储中文字符，需要在创建数据库的时候指定`字符集`

```
create database db_name default charset=utf8;
```

或者可以修改现有的表

```
alter table table_name convert to character set utf8;
```

# MySQL的一些特性

## MYSQL中的索引

使用索引可以在大型数据库中减少数据搜索的时间

常用的索引有INDEX，PRIMARY KEY（主键），FULLTEXT

### 1.创建INDEX索引

通过ALTER创建：

```
ALTER TABLE pet ADD INDEX(name(10));
```

该命令的意义为为`name`列创建索引，长度为10个字符（索引中只存储10个字符）

*通过索引进行检索比较复杂，日后再续*

### 2.主键

主键是每行都不同的`唯一`值，使用逐渐可以唯一的检索到某一行，在关系型数据库中一个表的主键通常会作为另一个表的外键`foreign key`

通过ALTER创建：

```
ALTER TABLE pet ADD PRIMARY KEY(id(10));
```

意为将`id`列创建为主键，注意id必须每一行都是唯一值

创建了主键之后使用`DESCRIBE pet`会看到`Key`这一列会发生变化

### 3.在创建表时创建索引

```sql
CREATE TABLE classics (
author VARCHAR(128),
title VARCHAR(128),
category VARCHAR(16),
year SMALLINT,
isbn CHAR(13),
INDEX(author(20)),
INDEX(title(20)),
INDEX(category(4)),
INDEX(year),
PRIMARY KEY (isbn)) ENGINE MyISAM;
```
注意最后要设置储存引擎，一般为`MyISAM`

## EXPLAIN方法

使用EXPLAIN方法可以解释如何发出的查询并打印出来，只要在SELECT前面加上它就能实现

```
EXPLAIN SELECT* FROM pet;
```
使用它也可以查看查询是否通过索引来实现
  
## 从文件中调用SQL命令

对于一些重复使用的命令，可以将其保存在文件中调用

### 在命令行中使用：

```
shell> mysql -h host -u user -p < finename
Enter password: ********
```

- 可以加上参数`-t`来使输出的格式与直接在命令行输入SQL命令时的输出格式一致
- 可以加上参数`-v`来打印所运行的SQL命令

### 在mysql中使用：
有两种方法

```
mysql> source filename;
mysql> \. filename
```

## 规范化

将数据分开放入表中并创建主键的过程称为规范化，主要目的是保证每一条信息在数据库中只出现一次

有三种范式

*涉及到好多图表，先把概念记到这里*

### 第一范式

第一范式有三个要求：
- 不能有包含相同类型数据的重复列出现
- 所有的列都是单值
- 要有一个主键来标识每一行

### 第二范式

第二范式首先要求满足第一范式，并在第一范式的基础上`消除多行间的冗余`

### 第三范式

第三范式在第一、第二范式的基础上，要求`数据不直接依赖于主键`

使用第三范式通常会增加表的数量，一般不需要使用

## 事务

使用MYSQL中的事务功能可以撤销一些操作，使用方法：

```
BEGIN;
UPDATE .... ;
INSERT .... ;
COMMIT;
```

在`BEGIN`之后的操作都是暂时性的，直到通过`COMMIT`提交之前他们都是可撤回的

撤回操作使用`ROLLBACK`

```
BEGIN;
UPDATE .... ;
INSERT .... ;
ROLLBACK;
```
这样UPDATE和INSERT操作就被撤回了

## 备份及恢复

使用`mysqldump`命令可以将数据库进行备份（在命令行下）

在备份之前最好将要备份的数据库加锁`LOCK`，或者确定备份过程中不会有用户向表中写入数据

```
LOCK TABLES .... ;
```
备份之后用`UNLOCK`解锁

```
UNLOCK TABLES;
```


mysqldump命令的使用方法：
```
mysqldump -u user -ppassword database
// database为要备份的数据库
```

直接运行该命令会将备份的内容打印在屏幕上，也可以将这些数据保存到文件中，使用`>`符号：

```
mysqldump -u user -ppassword database > database.sql
```

然后可以查看该文件，其中的内容包含所有重新创建表的命令和重新填充它们的数据

从备份文件中恢复数据库，使用`<`符号：

```
mysqldump -u user -ppassword -D database < database.sql
// -D表示恢复单个数据库
```
