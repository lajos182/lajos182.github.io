---
title: No.1 Python与数据库--MySql
date: 2020-05-12 13:40:50
tags: [Python, MySql]
toc: true
---

## 1.1 数据库简介

数据库简介：

- 用途：用于存储生活的几乎一切数据
- 概念：数据库服务器、数据库、数据表、一行数据(一条)、一列数据(字段)
- 分类：关系型数据库(`MySql`, `Oracle`, `SQL Server`)、非关系型数据库(`Redis`, `MongoDB`)
- `SQL`：`Structured Query Language`，结构化查询语言
- `SQL`分类：数据定义语言(`DDL`)、数据操作语言(`DML`)、数据查询语言(`DQL`)、数据控制语言(`DCL`)、数据事务语言(`DTL`)

<!--more-->

`RDBMS` (关系数据库管理系统(`Relational Database Management System`))术语:

- **数据库:** 数据库是一些关联表的集合。
- **数据表:** 表是数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。
- **列:** 一列(数据元素) 包含了相同类型的数据, 例如邮政编码的数据。
- **行：**一行（=元组，或记录）是一组相关的数据，例如一条用户订阅的数据。
- **冗余**：存储两倍数据，冗余降低了性能，但提高了数据的安全性。
- **主键**：主键是唯一的。一个数据表中只能包含一个主键。你可以使用主键来查询数据。
- **外键：**外键用于关联两个表。
- **复合键**：复合键（组合键）将多个列作为一个索引键，一般用于复合索引。
- **索引：**使用索引可快速访问数据库表中的特定信息。索引是对数据库表中一列或多列的值进行排序的一种结构。类似于书籍的目录。
- **参照完整性:** 参照的完整性要求关系中不允许引用不存在的实体。与实体完整性是关系模型必须满足的完整性约束条件，目的是保证数据的一致性。

`MySQL` 是一个关系型数据库管理系统，由瑞典 `MySQL AB `公司开发，目前属于 `Oracle `公司。`MySQL` 是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

## 1.2. `MySQL`(`Ubuntu`)的安装

+ 安装：`sudo apt-get install mysql-server`(若出现问题，应该删除自己添加的系统服务)

+ 安全配置：`sudo  mysql_secure_installation`

+ 连接测试：`mysql -h 主机 -u 用户名(默认root) -p`

+ 其他：端口3306；提出`\h`或`help`；所有命令都`;`结尾

## 1.3. 数据库定义语言(`DDL`)

![ 数据库定义语言(`DDL`)](https://gitee.com/lajos/image_bucket/raw/master/skill/DDL.png)

+ 数据类型：

  - 整型(`_int`)：
    - `tinyint`：有符号(-128, 127)，无符号(0, 255)，一个字节；
    - `smallint`：有符号(-32768,  32767)，无符号(0, 65535)，两个字节；
    - `mediumint`：3个字节，范围(-8388608~8388607)；
    - `int`：4个字节，范围(-2147483648~2147483647)；
    - `bigint`：8个字节，范围(+-9.22*10的18次方)；
    - 注意：`tinyint`(4)中的4并不表示存储的长度，只有字段指定`zerofill`是有用，如0002
  - 浮点型(`float`, `double`)：
    - `float(m, d)`：单精度浮点数，4个字节，`m`表示总位数，`d`表示小数位数；
    - `double(m, d)`：双精度浮点数，8个字节，`m`表示总位置，`d`表示小数位数
  - 定点数：`decimal(m, d)`：参数`m`<65是总个数，`d`<30且 `d`<`m`是小数位，以字符串的形式存储浮点数，用于金融领域等要求严格的场景
  - 字符串(`char`, `varchar`, `_text`)：
    - `char(n)`：固定长度，最多255个字符；
    - `varchar(n)`：固定长度，最多65535个长度；
    - `tinytext`：可变长度，最多255个字符；
    - `text`：可变长度，最多65535个字符；
    - `mediumtext`：可变长度，最多2的24次方-1个字符；
    - `longtext`：可变长度，最多2的32次方-1个字符
    - 注意：`char(n)`若存入字符数小于`n`，则以空格补于其后，查询之时再将空格去掉。所以char类型存储的字符串末尾不能有空格，`varchar`不限于此；
    - 注意：`char(n)`固定长度，`char(4)`不管是存入几个字符，都将占用4个字节，`varchar`是存入的实际字符数+1个字节（n<=255）或2个字节(n>255)，所以`varchar(4)`,存入3个字符将占用4个字节；
    - 注意：`char`类型的字符串检索速度要比`varchar`类型的快。
  - 二进制数据(`_Blob`)：
    - `_Blob`和`_text`存储方式不同，`_Text`以文本方式存储，英文存储区分大小写，而`_Blob`是以二进制方式存储，不分大小写；
    - `_Blob`存储的数据只能整体读出；
    - `_Text`可以指定字符集，`_Blob`不用指定字符集
  - 日期时间类型(`date`, `time`, `datetime`, `timestamp`)：
    - `date`：日期，以`YYYY-MM-DD`的格式显示，如'2019-11-19'；
    - `time`：时间，以`HH:MM:SS`的格式显示，如'15:44:36'；
    - `datetime`：日期时间，以`YYYY-MM-DD HH:MM:SS`的格式显示，如'2019-11-19 15:44:36'；
    - `timestamp`：以`YYYY-MM-DD`的格式显示，自动存储记录修改时间，数据会随其他字段修改的时候自动刷新

+ 数据类型的属性：

  - `NULL`：数据列可包含`NULL`值；
  - `NOT NULL`：数据列不包含`NULL`值；
  - `DEFAULT`：默认值；
  - `PRIMARY KEY`：主键；
  - `AUTO_INCREMENT`：自动递增，适用于整数类型；
  - `UNSIGNED`：无符号；
  - `CHARACTER SET name`：指定字符集

+ 数据库引擎：常见的`MySQL`数据库引擎有`InnoDB`，`MyIsam`，`Memory`，`Mrg_Myisam`，`Blackhole`等。

  - `InnoDB`：事务型存储引擎，提供了对数据库`ACID`事务的支持，并实现了`SQL`标准的**四种隔离级别**，具有行级锁定及外键支持，该引擎的设计目标便是处理大容量数据的数据库系统，`MySQL`在运行时`InnoDB`会在内存中建立缓冲池，用于缓存数据及索引；
  - `MyIsam`：MySQL主流引擎之一，不支持事务操作，不支持行锁及外键，效率较低；
  - `Memory(Heap)`：`MEMORY`类型的表访问非常得快，因为它的数据是放在内存中的，并且默认使用`HASH`索引，但是一旦服务关闭，表中的数据就会丢失掉；`HEAP`允许只驻留在内存里的临时表格。驻留在内存里让`HEAP`要比`ISAM`和`MYISAM`都快，但是它所管理的数据是不稳定的，而且如果在关机之前没有进行保存，那么所有的数据都会丢失;
  - `Mrg_MyIsam`：一个相同的可以被当作一个来用的`MyISAM`表的集合，它将`MyIsam`引擎的多个表聚合起来，但是它的内部没有数据，真正的数据依然是`MyIsam`引擎的表中，但是可以直接进行查询、删除更新等操作；
  - `Blackhole`：任何写入到此引擎的数据均会被丢弃掉，不做实际存储；`Select`语句的内容永远是空，它会丢弃所有的插入的数据，服务器会记录下`Blackhole`表的日志，所以可以用于复制数据到备份数据库。

+ 数据库索引：用于快速找出在某个列中有一特定值的行

  - 普通索引(`index()`)：`MySQL`中基本索引类型，没有什么限制，允许在定义索引的列中插入重复值和空值，纯粹为了查询数据更快一点；
  - 唯一索引(`unique()`)：索引列中的值必须是唯一的，但是允许为空值；
  - 主键索引(`primary key()`)：是一种特殊的唯一索引，不允许有空值；
  - 组合索引：在表中的多个字段组合上创建的索引，只有在查询条件中使用了这些字段的左边字段时，索引才会被使用，使用组合索引时遵循最左前缀集合；
  - 全文索引：只有在`MyISAM`引擎上才能使用，只能在`CHAR`, `VARCHAR`, `TEXT`类型字段上使用全文索引；
  - 空间索引：对空间数据类型的字段建立的索引，`MySQL`中的空间数据类型有四种，`GEOMETRY`、`POINT`、`LINESTRING`、`POLYGON`。在创建空间索引时，使用`SPATIAL`关键字。要求，引擎为`MyISAM`，创建空间索引的列，必须将其声明为`NOT NULL`。

很多选项都可以在创建表时指定，如：

```sql
create table user(
    id int auto_increment,
    name varchar(20),
    primary key(id),
    unique(name)
)engine=innodb default charset=utf8;
```

添加索引：

```sql
alter table user add index(em);				# 给em字段添加普通索引
alter table user add unique(username);		# 给username字段添加唯一索引
alter table user add primary key(id);		# 将id设置为主键索引
alter table user drop index em;				# 删除em字段的普通索引
```

## 1.4. 数据库操作语言(`DML`)

![数据库操作语言(`DML`)](https://gitee.com/lajos/image_bucket/raw/master/skill/DML.png)

+ 创建一张`star`表：

```sql
create table star(
    id int auto_increment,
    name varchar(20) not null,
    money float not null,
    province varchar(20) default null,
    age tinyint unsigned not null,
    sex tinyint not null,
    primary key(id)
)engine=innodb default charset=utf8;
```

+ 插入数据：

  - 方式1：不指定字段，按照数据表的数据添加一条数据的全部字段

  ```sql
  insert into star values(1,'黄晓明',200000,'山东',28,0);
  ```

  > 可以同时插入多条数据，每条一个小括号

  - 方式2：指定字段，只需要传递指定字段的值

  ```sql
  insert into star(name,money,age,sex,province) values('小岳岳',4000000, 33, 0, '河南')；
  ```

  > 可以同时插入多条数据，每条一个小括号

+ 查询数据：

```sql
select * from star;
```

+ 删除数据：删除操作一定不要忘了指定条件，否则后果自负。

```sql
delete from star where id = 1;
```

+ 修改数据：修改操作一定不要忘了指定条件，否则后果自负。

```sql
update star set age=22, money=8000000 where id=6;
```

## 1.5. 数据库查询语言(`DQL`)

![数据库查询语言(`DQL`)](https://gitee.com/lajos/image_bucket/raw/master/skill/DQL.png)

+ 基础查询：`select * from star;`

+ 指定字段查询：`select id,name,money from star;`

+ 排除重复记录：`select distinct name from star;`(使用`distinct`修饰的字段组合不能重复)

+ 指定条件查询：

```sql
select id,name,money from star where id > 4;
select id,name from star where id between 2 and 5;
select id,name from star where id in(2,4,6);
select id,name from star where name like '小%';
```

+ 结果集排序：`select id,name,money from star order by money desc;` 

  - `order by`：指定排序字段，`asc`表示升序，默认的，`desc`表示降序
  - 也可以多字段排序，先按照前面的字段排序，再按照后面字段排序。`select id,name,money,age from star order by money desc,age asc;`

+ 限制结果集：

```sql
select id,name,money from star limit 3;					# 取3条数据
select id,name,money from star limit 2,3;				# 偏移2条，然后取3条数据
select id,name,money from star limit 3 offset 2;		# 偏移2条，然后取3条数据
```

+ 常用的集合函数

| 函数  | 功能     |
| ----- | -------- |
| count | 统计个数 |
| sum   | 求和     |
| max   | 最大值   |
| min   | 最小值   |
| avg   | 平均值   |

> 1. 使用count时指定任何字段都行
> 2. 使用其他函数时必须要指定字段

```sql
select count(*) from star;
select max(money) as m from star;
```

> `as`可以给字段其别名

+ 分组操作

```sql
select * from star group by sex;					# 分组
select count(*), sex from star group by sex;		# 分组统计
```

+ 结果集过滤：

```sql
select count(*) as c,province from star group by province having c>1;
```

> 搜索所有记录，然后按照省份分组，统计成员大于1的省份

## 1.6. 数据控制语言(`DCL`)

![ 数据控制语言(`DCL`)](https://gitee.com/lajos/image_bucket/raw/master/skill/DCL.png)

## 1.7. 多表联合查询

+ 隐式内连接：没有出现`join`关键的连接, `select 字段 from 表1, 表2 where 关联条件;`

```sql
select username, name from user, goods where user.gid = goods.gid;
```

> 查看用户买的商品名

+ 显示内连接：会出现`join`关键字，后面的条件使用`on`，`select 字段 from 表1 [inner/cross] join 表2 on user.gid = goods.gid;`

```sql
select username, name from user [inner/cross] join goods on user.gid = goods.gid;
```

> 功能同上

+ 外左连接：以左表为主，`select 字段 from 表1 left [outer] join 表2 on 关联条件;`

```sql
select username, name from user left [outer] join goods on user.gid = goods.gid;
```

> 以左表为主，显示左边所有内容，右表不匹配的选项显示`NULL`

+ 外右连接：以右表为主，`select 字段 from 表1 right [outer] join 表2 on 关联条件;`

```sql
select username, name from user right [outer] join goods on user.gid = goods.gid;
```

> 以右表为主，显示右边所有内容，左表不匹配的选项显示`NULL`

## 1.8. 多表联合操作

+ 子(嵌套)查询操作

  - 说明：一条`sql`语句的查询结果作为另一条`sql`语句的条件
  - 示例：`select * from user where gid in (select gid from goods);`

+ 联合更新(同时更新多个表的数据)

  - 示例：`update user u, good g set u.gid=0, g.price=8000 where u.gid = g.gid and u.id=1`

## 1.9. 事务处理语言(`DTL`)

+ 事务：`MySQL`使用`INNODB`数据库引擎，用来维持数据库的完整性，常用来管理`insert`、`delete`、`update`

+ 开启事务：禁止自动提交，`set autocommit=0;`

+ 操作回滚：出现异常时使用，`rollback;`

+ 提交操作：没有异常，`commit;`

## 1.10. 外键索引(约束)

+ 所谓**外键**就是一个表的主键作为了另一个表的关联字段，然后在添加一点设置就成为了外键，可以简化多个关联表之间的操作，如：删除一个另一个跟着删除

示例：一个表示用户组的表`t_group`，一个用户表`t_user`

```sql
create table t_group(
    id int,
    name varchar(20),
    primary key(id)
);
create table t_user(
    id int,
    name varchar(20),
    groupid int,
    primary key(id),
    foreign key(groupid) references t_group(id) on delete restrict on update restrict
);
```

+ 约束行为：

  - `NO ACTION`或`RESTRICT`：删除或修改时不做任何操作，直接报错
  - `SET NULL`：删除或更新时，都设置为`null`
  - `CASCADE`：删除或更新时，同时删除或更新从表的数据

## 1.11. 数据清空(`DELETE`与`TRUNCATE`)

+ `delete from 表名;`：再次插入数据时，自增字段的值接着清空前的数据增加

+ `truncate table 表名;`：删除数据，同时将`AUTO_INCREMENT`的初始值设为1

## 1.12. 数据库的备份与恢复

+ 备份：`mysqldump -uroot -p test > test.sql`

+ 恢复：`mysql -uroot -p test < test.sql`

## 1.13. `Python`操作`MySql`

`Python`操作`MySQL`有5中方式，`MySQLdb`、`mysqlclient`、`PyMySQL`、`peewee`、`SQLAlchemy`，在`Python`的`Web`框架`Flask`和`Django`中都采用`orm`进行封。

### 1.13.1. `MySQLdb`

`MySQLdb`又叫`MySQL-python`，是`Python`连接`MySQL`最流行的一个驱动，很多框架都也是基于此库进行开发，遗憾的是它只支持`Python2.x`，而且安装的时候有很多前置条件，因为它是基于`C`开发的库，在`Windows`平台安装非常不友好，经常出现失败的情况，现在基本不推荐使用，取代的是它的衍生版本。

```shell
#  前置条件
sudo apt-get install python-dev libmysqlclient-dev  # Ubuntu
sudo yum install python-devel mysql-devel  # Red Hat / CentOS
# 安装
pip install Mysql-python
# Windwos直接通过exe文件安装
```

```python
import Mysqldb

db = Mysqldb.connect(
    host='localhost',  # 主机名
    user='root',  # 用户名
    passwd='pythontab.com',  # 密码
    db='testdb'  # 数据库名称
)
# 获取游标
cur = db.cursor()
# 执行的都是原生的SQL语句
cur.execute('select * from mytable;')
for row in cur.fetchall():
    print(row[0])
db.close()
```

### 1.13.2. `mysqlclient`

由于`MySQL-python(MySQLdb)`年久失修，后来出现了它的`Fork`版本`mysqlclient`，完全兼容`MySQLdb`，同时支持`Python3.x`，是`Django ORM`的依赖工具，如果你想使用原生`SQL`来操作数据库，那么推荐此驱动。安装方式和`MySQLdb`是一样的，`Windows`可以在`https://www.lfd.uci.edu/~gohlke/pythonlibs/#mysqlclient`网站找到对应版本的`whl`包下载安装。

+ 安装方式：`pip install mysqlclient`, 如果安装出现问题，需要安装`Python3.x-dev`、`libmysqlclient-dev`

### 1.13.3. `PyMySQL`

`PyMySQL`是纯`Python`实现的驱动，速度上比不上`MySQLdb`和`mysqlclient`，最大的特点可能就是它的安装方式没那么繁琐，同时也兼容`MySQL-python`

+ 安装方式：`pip install PyMySQL`，为了兼容`mysqldb`，只需要加入`pymysql.install_as_MySQLdb()`

```python
import pymysql

conn = pymysql.connect(host='127.0.0.1', user='root', passwd="pythontab.com", db='testdb')
cur = conn.cursor()
cur.execute("SELECT Host,User FROM user")
for r in cur:
    print(r)
cur.close()
conn.close()
```

### 1.13.4. `peewee`

写原生`SQL`的过程非常繁琐，代码重复，没有面向对象思维，继而诞生了很多封装`wrapper`包和`ORM`框架，`ORM`是`Python`对象与数据库关系表的一种映射关系，有了 `ORM`你不再需要写`SQL`语句。提高了写代码的速度，同时兼容多种数据库系统，如`sqlite`, `mysql`、`postgresql`，付出的代价可能就是性能上的一些损失。如果你对 `Django`自带的`ORM`熟悉的话，那么`peewee`的学习成本几乎为零。它是`Python`中是最流行的 `ORM` 框架。

+ 安装：`pip install peewee`

  - `MySQL`数据库连接：`db = MySQLDatabase(database='test', host='localhost', port=3306, user='root', passwd='jiang')`

  - `Sqlite`数据库连接：`db = SqliteDatabase(database=dbpath)`, `dbpath = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'test.db'`

  - 创建数据表：`模型类名.create_table()`

  - 插入数据：

    - 一般的`create`插入数据：`类名.create(username='lajos')`
    - 事务插入:

    ```python
    data = [
        {
            "username": "lajos",
            "age": 14
        },
        ...
    ]
    with db.atomic():
        for i in data:
            User.create(**i)
    ```

    - `insert_many()`插入多条数据：

     ```python
    data = [
        {
            "username": "lajos",
            "age": 14
        },
        ...
    ]
    with db.atomic():
        for i in range(0, len(data), 100)
            User.insert_many(data[i: i+100]).excute()
     ```

  - 查询数据：可以使用`get()`和`filter()`，也可以使用`select()`

```python
import datetime
from peewee import *

db = MySQLDatabase(database='test', host='localhost', port=3306, user='root', passwd='jiang')
db.connect()

class BaseModel(Model):

    class Meta:
        database = db

class User(BaseModel):
    username = CharField(unique=True)

class Tweet(BaseModel):
    user = ForeignKeyField(User, related_name='tweets')
    message = TextField()
    created_date = DateTimeField(default=datetime.datetime.now())
    is_published = BooleanField(default=True)

def create_table(table):
    # 如果不存在表就创建
    if not table.table_exists():
        table.create_table()

def drop_table(table):
    if table.table_exists():
        table.drop_table()


if __name__ == '__main__':
    # 创建表
    create_table(User)
    create_table(Tweet)
    # 插入数据
    data = [{
        'user_id': 1,
        'message': 'hello world'
    }]
    user = User.create(username='lajos')
    Tweet.create(user=user, message='lajos is a handsome man')
    with db.atomic():
        for i in data:
            Tweet.create(**i)
    # 插入大量数据时可以采用，速度快
    with db.atomic():
        for i in range(0, len(data), 100):
            Tweet.insert_many(data[i: i+100]).execute()
    # 查询数据
    user = User.get(username='lajos')
    User.filter()
```

### 1.13.5. `SQLAlchemy`

如果想找一种既支持原生`SQL`，又支持`ORM`的工具，那么`SQLAlchemy`是最好的选择，它非常接近`Java`中的`Hibernate`框架。安装使用`pip install sqlalchemy`

+ 原生`SQL`使用：

```python
import time
import threading
import sqlalchemy
from sqlalchemy import create_engine
from sqlalchemy.engine.base import Engine

conn_pool=create_engine( #创建sqlalchemy引擎
     "mysql+pymysql://webproject:web@192.168.1.18:3306/web?charset=utf8",  #数据库类型+数据库驱动://用户名:密码@主机:端口/数据库
     max_overflow=2, #超过连接池大小之后，允许最大扩展连接数，如果不指定默认为5；
     pool_size=5,   #连接池大小
     pool_timeout=30,#连接池如果没有连接了，最长等待时间
     pool_recycle=-1,#多久之后对连接池中连接进行一次回收

)



#多线程操作线程池
def task(arg):
    conn = conn_pool.raw_connection()
    cursor = conn.cursor()
    cursor.execute(
        #"select * from cmdb_worker_order"
        "select sleep(2)"
    )
    result = cursor.fetchall()
    cursor.close()
    conn.close()

if __name__ == '__main__'：
    for i in range(20):
        t = threading.Thread(target=task, args=(i,)) #5个线程 执行2秒 然后5个线程在去执行2秒
        t.start()
```

+ `ORM`使用

使用`ORM`方式创建数据表

```python
import datetime
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Text, ForeignKey, DateTime, UniqueConstraint, Index
from sqlalchemy.orm import relationship

Base = declarative_base()

class Users(Base):
    __tablename__ = 'users'                #表名称
    id = Column(Integer, primary_key=True) # primary_key=True设置主键
    name = Column(String(32), index=True, nullable=False) #index=True创建索引， nullable=False不为空。
    age = Column(Integer, default=18)        #数字字段
    email = Column(String(32), unique=True)  #设置唯一索引
    ctime = Column(DateTime, default=datetime.datetime.now) #设置默认值为当前时间（注意千万不要datetime.datetime.now()）
    extra = Column(Text, nullable=True)         #文本内容字段
    __table_args__ = (
        # UniqueConstraint('id', 'name', name='uix_id_name'), #设置联合唯一索引
        # Index('ix_id_name', 'name', 'extra'),                #设置联合索引
    )

class Hosts(Base):
    __tablename__ = 'hosts'

    id = Column(Integer, primary_key=True)
    name = Column(String(32), index=True)
    ctime = Column(DateTime, default=datetime.datetime.now)

# ##################### 一对多示例 #########################
class Hobby(Base):
    __tablename__ = 'hobby'
    id = Column(Integer, primary_key=True)
    caption = Column(String(50), default='篮球')

class Person(Base):
    __tablename__ = 'person'
    nid = Column(Integer, primary_key=True)
    name = Column(String(32), index=True, nullable=True)
    hobby_id = Column(Integer, ForeignKey("hobby.id"))
    # 与生成表结构无关，仅用于查询方便
    hobby = relationship("Hobby", backref='pers')

# ##################### 多对多示例 #########################

class Server2Group(Base):
    __tablename__ = 'server2group'
    id = Column(Integer, primary_key=True, autoincrement=True)
    server_id = Column(Integer, ForeignKey('server.id'))
    group_id = Column(Integer, ForeignKey('group.id'))

class Group(Base):
    __tablename__ = 'group'
    id = Column(Integer, primary_key=True)
    name = Column(String(64), unique=True, nullable=False)
    # 与生成表结构无关，仅用于查询方便
    servers = relationship('Server', secondary='server2group', backref='groups')

class Server(Base):
    __tablename__ = 'server'
    id = Column(Integer, primary_key=True, autoincrement=True)
    hostname = Column(String(64), unique=True, nullable=False)

def init_db():
    """
    根据类创建数据库表
    :return:
    """
    engine = create_engine(
        "mysql+pymysql://webproject:web@192.168.1.18:3306/web?charset=utf8",
        max_overflow=0,  # 超过连接池大小外最多创建的连接
        pool_size=5,  # 连接池大小
        pool_timeout=30,  # 池中没有线程最多等待的时间，否则报错
        pool_recycle=-1  # 多久之后对线程池中的线程进行一次连接的回收（重置）
    )

    Base.metadata.create_all(engine)


def drop_db(): #根据类 删除数据库表
    engine = create_engine(
        "mysql+pymysql://webproject:web@192.168.1.18:3306/web?charset=utf8",
        max_overflow=0,   # 超过连接池大小外最多创建的连接
        pool_size=5,      # 连接池大小
        pool_timeout=30,  # 池中没有线程最多等待的时间，否则报错
        pool_recycle=-1   # 多久之后对线程池中的线程进行一次连接的回收（重置）
    )

    Base.metadata.drop_all(engine) #这行代码很关键哦！！ 读取继承了Base类的所有表在数据库中进行删除表

if __name__ == '__main__':
    init_db()                      #执行创建
```

使用`ORM`方式操作数据库

```python
from SqlALchemy.models import Users
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine

engine = create_engine( "mysql+pymysql://webproject:web@192.168.1.18:3306/web?charset=utf8")
Session = sessionmaker(bind=engine)

# 每次执行数据库操作时，都需要创建一个会话
session = Session()

# ############# 执行ORM操作 #############

# 增加一条数据
obj1 = Users(username="张根"， age=18, email="1320168@qq.com", extra="handsome") #创建Users对象=1行数据
session.add(obj1)          #添加到表中

# 批量添加多条
session.add_all([
    Users(name="lajos", age=21, email="lajos@163.com", extra="merchant"),
    Users(name="刘峻轩", age=20, email="ljunxuan@163.com", extra="scientist")
])

# 删除操作
session.query(Users).filter(users.id==5).delete()

# 修改操作
session.query(Users).filter(Users.id==4).update({'name':'Martin'})
session.query(Users).filter(Users.id==4).update({Users.name: Users.name + "666"}, synchronize_session=False)#在原来的基础上做字符串
session.query(Users).filter(Users.id==18).update({Users.id: Users.id - 12},synchronize_session="evaluate")#在原来的基础上做数字+,-操作

# 查询
r0=session.query(Users).all()                             #查询user表中全部数据；
r1=session.query(Users).filter(Users.id > 2)              #查询user表中，id>2的记录；
r2=session.query(Users.age).all()                         #查询User表中的 age列； ##[(18,), (19,)]
r3=session.query(Users.age.label('cname')).all()          #使用别名查询

r4=session.query(Users).filter(Users.name=='Martin').all()      #查询姓名==Martin的用户
r5=session.query(Users).filter_by(name='Martin',age=19).all()   #filter_by方式查询
r6=session.query(Users).filter_by(name='Martin',age=19).first() #获取第 单个对象 print(r6.name)
r7=session.query(Users).filter(text("id<:value and name=:name")).params(value=18, name='Martin').order_by(Users.id).all() #查询 id>18 name=Martin的Users 根据 id排序，params支持传参数；
r8 = session.query(Users).from_statement(text("SELECT * FROM users where name=:name")).params(name='ed').all()#from_statement,申请使用原生sql

#条件查询
ret0 = session.query(Users).filter(Users.id.between(0,7), Users.name == 'Martin').all()  #between: 查询 user id在0--7之间，用户名为Martin 的数据;
ret1= session.query(Users).filter(Users.id.in_([1,6])).all()  #查询user_id in [1,6] 的数据
ret2 = session.query(Users).filter(~Users.id.in_([1,3,4])).all()  #查询user_id  not in [1,6] 的数据
ret = session.query(Users).filter(Users.id.in_(session.query(Users.id).filter_by(name='Martin'))).all()
#1.session.query(Users.id).filter_by(name='Martin') 查询name=Martin'的id
#2.在user表中 按1的结果 查询


# 多查询条件 逻辑运算、嵌套查询
from sqlalchemy import and_, or_
ret0 = session.query(Users).filter(and_(Users.id > 3, Users.name == 'eric')).all()
ret1 = session.query(Users).filter(or_(Users.id < 2, Users.name == 'eric')).all()
ret2 = session.query(Users).filter(
    or_(
        Users.id < 2,
        and_(Users.name == 'eric', Users.id > 3),
        Users.extra != ""
    )).all()

# 字符串、模糊匹配查询
ret0 = session.query(Users).filter(Users.name.like('M%')).first().name
ret1 = session.query(Users).filter(~Users.name.like('%r%')).first().name
print(ret0,ret1)

# 限制分页
ret = session.query(Users)[0:2]
print(ret)

# 排序
ret0 = session.query(Users).order_by(Users.id.desc()).all()                   #根据id,由大到小排序(desc).
ret1 = session.query(Users).order_by(Users.id.asc(),Users.age.desc()).all()   #根据id,由小到大排序(asc)，如id相等,由大到小排序(desc).
print([i.id for i in ret0])
print([i.id for i in ret1])

# group_by分组查询和having二次筛选
from sqlalchemy.sql import func

ret0 = session.query(Users).group_by(Users.age).all() #根据name字段进行分组

ret1 = session.query(     #使用name字段进行分组，求每组中 最大id 、最小id 、id 之和
    func.max(Users.id),
    func.sum(Users.id),
    func.min(Users.id)
).group_by(Users.name).all()


ret2 = session.query(
    func.max(Users.id),
    func.sum(Users.id),                          #对分组之后的数据进行 having筛选，
    func.min(Users.id)
).group_by(Users.name).having(func.min(Users.id) >2).all()
'''
having的作用：
例如：查询公司中 部门人数大于3人的部门，先按照部门分组，然后求人数，然后再having 大于3的；

'''

# 连表查询
ret = session.query(models.Users).join(models.Hobby, models.Users.id == models.Hobby.id,isouter=True).filter(models.Users.id >1)
#2个没有外键关系的表 做连表查询
print(ret)

ret = session.query(models.Person).join(models.Hobby).all()              #inin_join
print(ret)
ret = session.query(models.Person).join(models.Hobby,isouter=True).all() #left_join 调换位置 更换为 right_join
print(ret)

''''
left join：以左表为准，显示符合搜索条件的记录；

aID　　　　　aNum　　　　　bID　　　　　bName
　　　　  a20050115　　　 NULL　　　　　NULL


right join：以右表为准，显示符合搜索条件的记录；

aID　　　　　aNum　　　　　bID　　　　　bName
NULL　　　　 NULL　　　　　 8　　　　2006032408


inin_join：并不以谁为基础,它只显示符合条件
aID　　　　　aNum　　　　　bID　　　　　bName


'''

# 连表方式1：指定字段获取
persons=session.query(models.Person.name,models.Hobby.caption.label('hobby_caption')).join(models.Hobby)
for row in  persons:
    print(row.name,row.hobby_caption,)

# 连表方式2：获取所有字段
persons = session.query(models.Person, models.Hobby).join(models.Hobby)
for row in persons:
    #print(row)=2个对象
    print(row[0].name,row[1].caption)

# 连表方式3：relationship 连表查询
persons = session.query(models.Person).all()
for row in persons:
    print(row.name,row.hobby.caption)

'''
 hobby = relationship("Hobby", backref='pers')
 Hobby：正向查询
 backref：反向查询

ps:查询喜欢姑娘的所有人
hobby_obj=session.query(models.Hobby).filter(models.Hobby.id==2).first()
print(hobby_obj.pers)
'''

################################relationship增加######################
# 1.relationship正向增加
person_obj=models.Person(name='Tony',hobby=models.Hobby(caption='any'))


# 2.relationship 反向增加
hobby_obj=models.Hobby(caption='人妖')
hobby_obj.pers=[
        models.Person(name='李渊'),
        models.Person(name='西门庆'),
    ]
session.add(person_obj,hobby_obj)

基于relationship 做1对多操作


#############################多对多添加#################################

#添加方式1：同时添加男、女对象，直接添加相亲表；前提是 知道新添加男、女对象的ID；
session.add_all([
    models.Boy(name='张无忌'),
    models.Boy(name='宋青书'),
    models.Girl(name='周芷若'),
    models.Girl(name='赵敏'),
])
session.commit()

s2g =models.G2B(girl_id=1,boy_id=1)
session.add(s2g)
session.commit()

#添加方式2：通过女生对象 添加 相亲记录表
girl_obj = models.Girl(name='灭绝师太')                                     #创建1位女性朋友 灭绝
girl_obj.boys = [models.Boy(name='张三丰'),models.Boy(name='方正大师')]    #然后灭绝和 张三丰、方正大师分别相了1次亲
session.add(girl_obj)
session.commit()

##添加方式3：通过男生对象 添加 相亲记录
boy_obj = session.query(models.Boy).filter(models.Boy.name=='尹志平').first()  #创建1位男性朋友 尹志平
boy_obj.girls = [models.Girl(name='小龙女'),models.Girl(name='黄蓉')]         #然后尹志平 和小龙女、黄蓉分别 相了一次亲
session.add(boy_obj)
session.commit()

##################################多对多查询###################################

#boys = relationship('Boy', secondary='g2b', backref='girls')
#1.基于 relationship 正向查询
mie_jue = session.query(models.Girl).filter(models.Girl.name=='灭绝师太').first()
print( [i.name for i in mie_jue.boys]) #['方正大师', '张三丰']

#2.基于 relationship 反向查询
yi_zhi_ping = session.query(models.Boy).filter(models.Boy.name=='尹志平').first()
print( [i.name for i in yi_zhi_ping.girls]) #['小龙女', '黄蓉']


# 提交事务，上面的每次操作都要执行
session.commit()
# 关闭session
session.close()
```

> `filter`与`filter_by`的区别：
> `filter`传入的参数要用这样的格式：类名.列明，判断要用`==`表示，`session.query(Users).filter(Users.name =='Martin').filter(Users.age == 19).all()`;
> `filter_by`使用的语法和`Python`更接近，`session.query(Users).filter(name='Martin', age=19).all()`

### 1.13.6. `Flask-SQLAlchemy`

+ `SQLAlchemy`是一个关系型数据库框架，它提供了高层的`ORM`和底层的原生数据库的操作，让开发者不用直接和`SQL`语句打交道，而是通过`Python`对象来操作数据库，在舍弃一些性能开销的同时，换来的是开发效率的较大提升。一句话：就是对数据库的抽象！

+ `Flask-SQLAlchemy`是一个简化了`SQLAlchemy`操作的`flask`扩展，是`SQLAlchemy`的具体实现，封装了对数据库的基本操作。举例：如果说动物园是`SQLAlchemy`，那`Flask-SQLAlchemy`只是其中的一只。

## 1.14 常见问题

+ `Linux 20.04`采用`sudo apt-get install mysql-server`时，安装的版本其实是`mysql 8.0`，安装成功后无法使用`root`账户进入数据库？

  （1）查看默认配置文件：

  ```shell
  sudo cat /etc/mysql/debian.cnf
  # Automatically generated for Debian scripts. DO NOT TOUCH!
  [client]
  host     = localhost
  user     = debian-sys-maint
  password = qe7CxSdH2HczVtwL
  socket   = /var/run/mysqld/mysqld.sock
  [mysql_upgrade]
  host     = localhost
  user     = debian-sys-maint
  password = qe7CxSdH2HczVtwL
  socket   = /var/run/mysqld/mysqld.sock
  ```

  （2）以默认配置登录`mysql`,`mysql -udebian-sys-maint -p`，然后输入密码`qe7CxSdH2HczVtwL`

  （3）更改`root`密码：

  ```shell
  use mysql;
  # 下一行，密码改为了yourpassword，可以设置成其他的
  update mysql.user set authentication_string=password('yourpassword') where user='root' and Host ='localhost';
  update user set plugin="mysql_native_password"; 
  flush privileges;
  quit;
  ```

  （4）重启`mysql`：`sudo service mysql restart`

+ `Mysql`数据库远程主机无法连接是什么问题？

  （1）需要开启用户的访问权限

  ```mysql
  mysql –uroot –p
  use mysql;
  update user set host = '%' where user = 'test';
  grant all privileges on test.* to 'test'@'%' identified by '123456';
  flush privileges;
  quit;
  ```

  （2）需要修改`mysqld.cnf`中的`ip`绑定

  ```shell
  sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
  bind-address = 0.0.0.0
  # 保存退出
  # 重启服务
  sudo service mysql restart
  ```

  （3）进入服务器后台，添加相应的安全策略组规则（入方向），例如阿里云服务器后台配置：

| 授权策略 | 优先级 | 协议类型  | 端口范围        | 授权对象      | 描述            |
| -------- | ------ | --------- | --------------- | ------------- | --------------- |
| 允许     | 1      | 自定义TCP | 目的：3306/3306 | 源：0.0.0.0/0 | MySQL数据库连接 |