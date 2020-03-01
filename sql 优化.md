# SQL索引优化学习总结

[TOC]

## 一、Ubuntu 下mysql安装、启停

### 1.1 环境

MySQL 5.7.28 (5.7.28-0ubuntu0.16.04.2)

Ubuntu 18.04 LTS

### 1.2 安装
1. Ubuntu安装MySQL：

```
sudo apt-get install mysql-server mysql-client
```

2. 查看MySQL是否正常运行

```
ps aux|grep mysql
```

3. MySQL 启动、停止、重启

```
启动mysql应用： service mysql start
关闭mysql应用： service mysql stop
重启mysql应用： service mysql restart
```

4. MySQL 登陆

```
mysql -u root -p
```

## 二、配置编码

```
目录：
	/var/lib/mysql :mysql 数据库位置
	/etc/mysql/mysql.conf.d/mysqld.cnf:  配置文件

默认端口3306

查看编码：
	sql:show variables like '%char%';
	
设置编码：
	vi /etc/mysql/mysql.conf.d/mysqld.cnf
    为避免配置文件出错，在最后添加如下三个配置
	[client]
	default-character-set=utf8
	
	[mysqld]
	character_set_server=utf8
	collation_server=utf8_general_ci

重启 Mysql: service mysql restart
```



## 三、存储引擎

### 3.1 存储引擎
- InnoDB(默认) ：事务优先 （适合高并发操作，行锁）

查询数据库支持哪些引擎

```mysql
mysql>show engines;

可以看到innoDB是默认的存储引擎
```

查看当前使用的引擎

```mysql
mysql>show variables like '%storage_engine%';
```
## 四、SQL解析过程、索引、B树

### 4.1 SQL的编写与解析过程

```mysql
a.编写过程：
	select dinstinct  ..from  ..join ..on ..where ..group by ...having ..order by ..limit ..
	
b.解析过程：
	from .. on.. join ..where ..group by ....having ...select dinstinct ..order by limit ...
```

### 4.2 索引的利弊

常说的SQL优化，主要就是优化索引

- 索引： 相当于字典的目录，有了索引，可以很轻松的从字典中查到想要的数据

- 索引： index(索引)是帮助MySQL高效获取数据的数据结构，索引是数据结构，比如B+树，查询速度极快



**索引的弊端：**

1. 索引本身很大， 可以存放在内存/硬盘（通常为 硬盘）
2. 索引不是所有情况均适用：以下情况不建议建立索引 
   - 少量数据 
   -  频繁更新的字段  
   - 很少使用的字段
3. 索引会降低增删改的效率（即经常增删改的表不适合建立索引）

**索引的优势：**

1. 提高查询效率（降低IO使用率）
2. 降低CPU使用率 （...order by age desc，因为树索引本身就是一个排好序的结构，因此在排序时可以直接使用）

## 五、B+树与索引

### 5.1 B+树结构
只有叶子节点存放数据，其他存放索引地址

![image](http://hxraid.iteye.com/upload/picture/pic/56353/0988ccff-f845-36b3-804d-e57b93e1c5e6.jpg)

### 5.2 索引
**索引分类**：
- 主键索引：不能重复。id    不能是null（primary）；
- 唯一索引：不能重复。id    可以是null；
- 单值索引：单个列作为索引；
- 复合索引：多个列构成的索引，比如想在字典中查找王(wang)，那么先找w，再找a，再找n，再找g，这里w，a，n，g构成复合索引。

**创建索引：**

方式一

```mysql
create 索引类型  索引名  on 表(字段)
单值：
create index 索引名 on 表名(字段);
唯一：
create unique index 索引名 on 表名(字段) ;
复合索引
create index 索引名 on 表名(字段1,字段2);
```

方式二
```mysql
alter table 表名 索引类型  索引名（字段）
单值：
alter table 表名 add index 索引名(字段) ;
唯一：
alter table 表名 add unique index 索引名(字段);
复合索引
alter table 表名 add index 索引名(字段1,字段2);

注意：如果一个字段是primary key，则改字段默认就是 主键索引，比如id字段，通常为主键索引
```


删除索引：


```mysql
drop index 索引名 on 表名;
```

查询索引：

```mysql
show index from 表名 ;
show index from 表名 \G
```

## 六、SQL优化准备
### 6.1 SQL性能优化explain

- 分析SQL的执行计划 : explain 可以模拟SQL优化器执行SQL语句，从而知道SQL到底是怎么执行的


查询执行计划：  explain +SQL语句

```mysql

mysql> explain select * from course;
+----+-------------+--------+------------+------+---------------+------+---------+------+--------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra |
+----+-------------+--------+------------+------+---------------+------+---------+------+--------+----------+-------+
|  1 | SIMPLE      | course | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 998136 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+------+---------+------+--------+----------+-------+
1 row in set, 1 warning (0.00 sec)


 id：编号				
 select_type：查询类型
 table：表
 type：类型
 possible_keys ：预测用到的索引 
 key：实际使用的索引
 key_len：实际使用索引的长度
 ref：表之间的引用
 rows：通过索引查询到的数据量 
 Extra：额外的信息
```
### 6.2 准备数据

>  为了效率插入数据，创建表，插入测试数据，均使用python完成，代码如下


```python
# -*- coding: utf-8 -*-#

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import string
import random
from functools import wraps
import time


app = Flask(__name__)


DB_DIALECT = 'mysql'
DB_DRIVER = 'mysqldb'
DB_USERNAME = 'root'
DB_PASSWORD = 'xxx'
DB_HOST = '127.0.0.1'
DB_PORT = '3306'
DB_DATABASE = 'sql_optimize'

SQLALCHEMY_DATABASE_URI: str = '{}+{}://{}:{}@{}:{}/{}?charset=utf8'.format(DB_DIALECT, DB_DRIVER,
                                                                            DB_USERNAME, DB_PASSWORD,
                                                                            DB_HOST, DB_PORT,
                                                                            DB_DATABASE)

app.config['SQLALCHEMY_DATABASE_URI'] = SQLALCHEMY_DATABASE_URI
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)


class Course(db.Model):
    __tablename__ = 'course'
    cid = db.Column(db.Integer, primary_key=True, autoincrement=True)
    cname = db.Column(db.String(20))
    tid = db.Column(db.Integer)


class Teacher (db.Model):
    __tablename__ = 'teacher'
    tid = db.Column(db.Integer, primary_key=True, autoincrement=True)
    tname = db.Column(db.String(20))
    tcid = db.Column(db.Integer)


class TeacherCard (db.Model):
    __tablename__ = 'teacherCard'
    tcid = db.Column(db.Integer, primary_key=True, autoincrement=True)
    tcdesc = db.Column(db.String(200))


# 时间装饰器
def time_cal(func):
    @wraps(func)
    def inner():
        start = time.perf_counter()
        func()
        end = time.perf_counter()
        print('总共花了%f秒' % (end-start))
    return inner


# 清空所有表
def drop_tables():
    db.drop_all()
    print('Drop tables finished!')


# 创建所有表
def create_tables():
    db.create_all()
    print('Create tables finished!')


# 创建测试数据
# 数据各4条
@time_cal
def create_course_data():
    cname_l = ['python', 'sql', 'c++', 'javascript']
    tid_l = [1, 2, 3, 4]
    for (cname, tid) in zip(cname_l, tid_l):
        course = Course(cname=cname, tid=tid)
        db.session.add(course)
    db.session.commit()


@time_cal
def create_teacher_data():
    tname_l = ['a', 'b', 'c', 'd']
    tcid_l = [1, 2, 3, 4]
    for (tname, tcid) in zip(tname_l, tcid_l):
        teacher = Teacher(tname=tname, tcid=tcid)
        db.session.add(teacher)
    db.session.commit()


@time_cal
def create_teacher_card_data():
    for i in range(4):
        t_c = TeacherCard()
        db.session.add(t_c)
    db.session.commit()


if __name__ == '__main__':
    drop_tables()
    create_tables()
    create_course_data()
    create_teacher_data()
    create_teacher_card_data()


然后执行如下
python37 sql_data_insertion.py

Drop tables finished!
Create tables finished!
总共花了0.090447秒
总共花了0.083834秒
总共花了0.083073秒

```
上述代码执行完后，总共创建了三张表，结构分别如下


```sql

mysql> desc course;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| cid   | int(11)     | NO   | PRI | NULL    | auto_increment |
| cname | varchar(20) | YES  |     | NULL    |                |
| tid   | int(11)     | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)


mysql> desc teacher;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| tid   | int(11)     | NO   | PRI | NULL    | auto_increment |
| tname | varchar(20) | YES  |     | NULL    |                |
| tcid  | int(11)     | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)



mysql> desc teacherCard;
+--------+--------------+------+-----+---------+----------------+
| Field  | Type         | Null | Key | Default | Extra          |
+--------+--------------+------+-----+---------+----------------+
| tcid   | int(11)      | NO   | PRI | NULL    | auto_increment |
| tcdesc | varchar(200) | YES  |     | NULL    |                |
+--------+--------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)


其中，course.tid和teacher.tid为外键约束，teacher.tcid和teacherCard.tcid为外键约束

```




### 6.3 explain中输出各字段含义

#### 1. Id 编号 

查询课程编号为2  或 教师证编号为3  的老师信息



```
explain select t.* from teacher t, course c, teacherCard tc
where t.tid = c.tid
and t.tcid = tc.tcid
and (c.cid = 2 or tc.tcid = 3 );
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                 | rows | filtered | Extra                                              |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | c     | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                |    4 |   100.00 | NULL                                               |
|  1 | SIMPLE      | t     | NULL       | ALL    | PRIMARY       | NULL    | NULL    | NULL                |    4 |    25.00 | Using where; Using join buffer (Block Nested Loop) |
|  1 | SIMPLE      | tc    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | sql_optimize.t.tcid |    1 |   100.00 | Using index                                        |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+----------------------------------------------------+


```

explain + sql:

id值相同，从上往顺序执行，注意这里说的id指的是explain输出的ID


表的执行顺序因数量的个数改变而改变的原因： 笛卡儿积

**数据小的表优先查询，执行SQL结果也有可能不一定是该结果，原因是服务层中有SQL优化器，可能会影响我们的的结果**



**ID值不同：ID值越大越优先查询 (本质：在嵌套子查询时，先查内层再查外层)，注意这里说的ID是explain出来的ID**

查询教授SQL课程的老师的描述（desc）

```mysql
explain select tc.tcdesc from teacherCard tc,course c,teacher t
where c.tid = t.tid
and t.tcid = tc.tcid
and c.cname = 'sql';


+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                 | rows | filtered | Extra       |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
|  1 | SIMPLE      | c     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                |    4 |    25.00 | Using where |
|  1 | SIMPLE      | t     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | sql_optimize.c.tid  |    1 |   100.00 | Using where |
|  1 | SIMPLE      | tc    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | sql_optimize.t.tcid |    1 |   100.00 | NULL        |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)


```

将以上多表查询转为子查询形式：

```mysql

explain select tc.tcdesc from teacherCard tc where tc.tcid =
(select t.tcid from teacher t where t.tid =
(select c.tid from course c where c.cname = 'sql')
);

+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | tc    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  2 | SUBQUERY    | t     | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  3 | SUBQUERY    | c     | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  |    4 |    25.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
3 rows in set, 1 warning (0.02 sec)


```



子查询+多表： 

```mysql
explain select t.tname ,tc.tcdesc from teacher t,teacherCard tc
where t.tcid= tc.tcid
and t.tid = (select c.tid from course c where cname = 'sql') ;

+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | t     | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  1 | PRIMARY     | tc    | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  2 | SUBQUERY    | c     | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  |    4 |    25.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)

```

**ID值有相同，又有不同： ID值越大越优先；ID值相同，从上往下 顺序执行**

#### 2. select_type 查询类型

select_type 是查询类型

- PRIMARY：包含子查询SQL中的主查询 （最外层）
- SUBQUERY：包含子查询SQL中的 子查询 （非最外层）
- simple：简单查询（不包含子查询、union）
- derived：衍生查询（使用到了临时表）


a.在from子查询中只有一张表

```mysql
explain select  cr.cname from ( select * from course where tid in (1,2) ) cr;


+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | course | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    4 |    50.00 | Using where |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

```


b.在from子查询中， 如果有table1 union table2 ，则table1就是derived（衍生表）, table2就是union

```mysql
explain select cr.cname from (select * from course where tid = 1 union select * from course where tid = 2 ) cr;

+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type  | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | PRIMARY      | <derived2> | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    4 |   100.00 | NULL            |
|  2 | DERIVED      | course     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    4 |    25.00 | Using where     |
|  3 | UNION        | course     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    4 |    25.00 | Using where     |
| NULL | UNION RESULT | <union2,3> | NULL       | ALL  | NULL          | NULL | NULL    | NULL | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+

```


union:上例
union result :告知开发人员，那些表之间存在union查询，即2和3存在union查询




#### 3. type 索引类型

type索引索引类型

> NULL > system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

我们常见的索引类型有情况有

> system>const>eq_ref>ref>range>index>all

**要对type进行优化的前提：有索引**，其中：system，const只是理想情况，我们优化常常实际能达到 ref > range



**system（忽略）: 只有一条数据的`系统表` ；或`衍生表`只有一条数据的主查询**

```mysql
create table test01(
    tid int(3),
    tname varchar(20)
);
insert into test01 values(1,'a');
```


增加索引

```mysql
alter table test01 add primary key(tid);
```


**const：仅仅能查到一条数据的SQL ，用于Primary key 或unique索引  （类型与索引类型有关）**

```mysql
alter table test01 add constraint tid_pk primary key(tid);  #有主键忽略
explain select tid from test01 where tid =1;
```


**eq_ref：唯一性索引，对于每个索引键的查询，返回匹配唯一行数据（有且只有1个，不能多 、不能为0）**

```mysql
select ... from ..where name = ... .常见于唯一索引 和主键索引。
alter table teacherCard add constraint pk_tcid primary key(tcid);
alter table teacher add constraint uk_tcid unique index(tcid) ;
explain select t.tcid from teacher t,teacherCard tc where t.tcid = tc.tcid ;


+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+--------------------------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                 | rows | filtered | Extra                    |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+--------------------------+
|  1 | SIMPLE      | t     | NULL       | index  | uk_tcid       | uk_tcid | 5       | NULL                |    4 |   100.00 | Using where; Using index |
|  1 | SIMPLE      | tc    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | sql_optimize.t.tcid |    1 |   100.00 | Using index              |
+----+-------------+-------+------------+--------+---------------+---------+---------+---------------------+------+----------+--------------------------+
2 rows in set, 1 warning (0.00 sec)

```


以上SQL，用到的索引是 t.tcid,即teacher表中的tcid字段，如果teacher表的数据个数和连接查询的数据个数一致（都是3条数据），则有可能满足eq_ref级别；否则无法满足。



**ref：非唯一性索引，对于每个索引键的查询，返回匹配的所有行（0，多）**
准备数据：

```mysql
insert into teacher values('a', 5)；
alter table teacher add index idx_name(tname);
```

测试：

```mysql
explain select * from teacher where tname = 'b';

+----+-------------+---------+------------+------+---------------+----------+---------+-------+------+----------+-------+
| id | select_type | table   | partitions | type | possible_keys | key      | key_len | ref   | rows | filtered | Extra |
+----+-------------+---------+------------+------+---------------+----------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | teacher | NULL       | ref  | idx_name      | idx_name | 63      | const |    2 |   100.00 | NULL  |
+----+-------------+---------+------------+------+---------------+----------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

```



**range：检索指定范围的行 ,where后面是一个范围查询(between   ,> < >=,     特殊:in有时候会失效 ，从而转为无索引all)**

```mysql
explain select t.* from teacher t where t.tid in (1,2);

+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t     | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |    2 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)


explain select t.* from teacher t where t.tid <3;

+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t     | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |    2 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)


```



**index：查询全部索引中数据**
explain select tid from teacher ;  --tid 是索引，只需要扫描索引表，不需要所有表中的所有数据，这也称之为覆盖索引

如果一个索引包含(或覆盖)所有需要查询的字段的值，称为‘覆盖索引’。即只需扫描索引而无须回表。

```mysql
explain select tid from teacher ;

+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | teacher | NULL       | index | NULL          | uk_tcid | 5       | NULL |    5 |   100.00 | Using index |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)


```



**all：查询全部表中的数据**
explain select tid from course ;  --tid不是索引，需要全表所有，即需要所有表中的所有数据

```mysql
mysql> explain select cid from course \G;

+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | course | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    4 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

```



**小结：**

system/const：结果只有一条数据
eq_ref：结果多条，但是每条数据是唯一的 ；
ref：结果多条，但是每条数据是是0或多条 ；

#### 4. possible_keys 可能用到的索引


possible_keys 指可能用到的索引，是一种预测，不准

```mysql
alter table course add index idx_cname(cname);

explain select t.tname ,tc.tcdesc from teacher t,teacherCard tc
where t.tcid= tc.tcid
and t.tid = (select c.tid from course c where cname = 'sql') ;

+----+-------------+-------+------------+-------+-----------------+-----------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys   | key       | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+-----------------+-----------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | t     | NULL       | const | PRIMARY,uk_tcid | PRIMARY   | 4       | const |    1 |   100.00 | NULL  |
|  1 | PRIMARY     | tc    | NULL       | const | PRIMARY         | PRIMARY   | 4       | const |    1 |   100.00 | NULL  |
|  2 | SUBQUERY    | c     | NULL       | ref   | idx_cname       | idx_cname | 63      | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+-----------------+-----------+---------+-------+------+----------+-------+
3 rows in set, 1 warning (0.00 sec)

```


如果 possible_key/key是NULL，则说明没用索引

#### 5. key 实际使用到的索引

key 指的实际使用到的索引


#### 6. key_len 索引的长度 

**key_len的计算**
1. varchar（20）--> 20*3(utf8) + 1(允许空值) + 2（可变长度）
2. int是4个字节


utf8:1个字符3个字节
gbk:1个字符2个字节
latin:1个字符1个字节

key_len 指的是索引的长度 ，用于判断复合索引是否被完全使用  （a,b,c)，当使用复合索引的时候，索引长度为每个单独索引长度的累加和。


#### 7. ref  指明当前表所参照的字段

ref 注意与type中的ref值区分，指明当前表所参照的字段
select ....where a.c = b.x ;  (其中b.x可以是常量，const)

```mysql
alter table course add index idx_tid(tid);
explain select * from course c,teacher t where c.tid = t.tid  and t.tname ='a';


+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys    | key      | key_len | ref                | rows | filtered | Extra |
+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
|  1 | SIMPLE      | t     | NULL       | ref  | PRIMARY,idx_name | idx_name | 63      | const              |    2 |   100.00 | NULL  |
|  1 | SIMPLE      | c     | NULL       | ref  | idx_tid          | idx_tid  | 5       | sql_optimize.t.tid |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)


第一条记录ref: const，t.tname ='a'，即当前表t参照了常量字段
第二条记录ref: sql_optimize.t.tid，c.tid = t.tid，即当前表c参照了t.tid字段
```

#### 8. rows 索引优化查询的数据个数

rows指被索引优化查询的数据个数 (实际通过索引而查询到的数据个数)

```mysql

# t表中有两个tname='a', 所以t这一行rows=2

explain select * from course c,teacher t where c.tid = t.tid  and t.tname ='a';

+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys    | key      | key_len | ref                | rows | filtered | Extra |
+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
|  1 | SIMPLE      | t     | NULL       | ref  | PRIMARY,idx_name | idx_name | 63      | const              |    2 |   100.00 | NULL  |
|  1 | SIMPLE      | c     | NULL       | ref  | idx_tid          | idx_tid  | 5       | sql_optimize.t.tid |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+


# t表中只有一个tname='b', 所以t这一行rows=1

explain select * from course c,teacher t where c.tid = t.tid  and t.tname ='b';


+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys    | key      | key_len | ref                | rows | filtered | Extra |
+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
|  1 | SIMPLE      | t     | NULL       | ref  | PRIMARY,idx_name | idx_name | 63      | const              |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | c     | NULL       | ref  | idx_tid          | idx_tid  | 5       | sql_optimize.t.tid |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+------------------+----------+---------+--------------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)

```

#### 9. Extra 

Extra：
- using filesort：性能消耗大，需要“额外”的一次排序（查询），常见于 order by 语句中。



10个人根据年龄排序，a1:姓名  a2：年龄

```mysql
create table test02(
    a1 char(3),
    a2 char(3),
    a3 char(3),
    index idx_a1(a1),
    index idx_a2(a2),
    index idx_a3(a3)
);
explain select * from test02 where a1 ='' order by a1;


+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
| id | select_type | table  | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | test02 | NULL       | ref  | idx_a1        | idx_a1 | 10      | const |    1 |   100.00 | NULL  |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

```



a1:姓名  a2：年龄

```mysql
# 索引走a1但是排序根据a2，那么会出现filesort

explain select * from test02 where a1 ='' order by a2;

+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+---------------------------------------+
| id | select_type | table  | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra                                 |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+---------------------------------------+
|  1 | SIMPLE      | test02 | NULL       | ref  | idx_a1        | idx_a1 | 10      | const |    1 |   100.00 | Using index condition; Using filesort |
+----+-------------+--------+------------+------+---------------+--------+---------+-------+------+----------+---------------------------------------+
1 row in set, 1 warning (0.03 sec)

```

**小结：**

**对于单索引， 如果排序和查找是同一个字段，则不会出现using filesort，如果排序和查找不是同一个字段，则会出现using filesort**
**怎么避免： where哪些字段，就order by哪些字段**



复合索引：不能跨列（最佳左前缀）

```mysql
drop index idx_a1 on test02;
drop index idx_a2 on test02;
drop index idx_a3 on test02;

alter table test02 add index idx_a1_a2_a3 (a1,a2,a3) ;

explain select *from test02 where a1='' order by a3; --using filesort
explain select *from test02 where a2='' order by a3; --using filesort
explain select *from test02 where a1='' order by a2;
explain select *from test02 where a2='' order by a1; --using filesort
```

**小结：避免： where和order by 按照复合索引的顺序使用，不要跨列或无序使用（最佳左前缀），如果where中的使用到索引字段顺序+order by 字段顺序没有跨列，也算不跨列**



- using temporary：性能损耗大 ，用到了临时表，一般出现在group by 语句中，表示已经有表了，但不适用，必须再来一张表

SQL解析过程：
from .. on.. join ..where ..group by ....having ...select dinstinct ..order by limit ...

```mysql
explain select a1 from test02 where a1 in ('1','2','3') group by a1;
explain select a1 from test02 where a1 in ('1','2','3') group by a2; --using temporary
```

**避免：查询那些列，就根据那些列 group by**

上边第二句会在MySQL5.7上报错

> ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'caojx.test02.a1' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

主要是sql_mode 中有一个 only_full_group_by 的值，参考如下两篇文章解决一下

https://blog.csdn.net/jgj0129/article/details/53420574

https://www.cnblogs.com/zhangzhiqin/p/8507182.html



-  using index ：性能提升，索引覆盖（覆盖索引）

原因：不读取原文件，只从索引文件中获取数据 （不需要回表查询）

只要使用到的列全部都在索引中，就是索引覆盖using index

例如：test02表中有一个复合索引(a1,a2,a3)

```mysql
explain select a1,a2 from test02 where a1='' or a2= ''; --Using where; Using index（使用or造成回表查询）
explain select a1,a2 from test02 where a1='' and a2= ''; --Using index

drop index idx_a1_a2_a3 on test02;
alter table test02 add index idx_a1_a2(a1,a2) ;

explain select a1,a3 from test02 where a1='' or a3= '';


+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | test02 | NULL       | ALL  | idx_a1_a2     | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

```

注意：如果用到了索引覆盖(using index时)，会对 possible_keys和key造成影响
a.如果没有where，则索引只出现在key中；
b.如果有where，则索引 出现在key和possible_keys中

```mysql
mysql> explain select a1,a2 from test02 where a1='' or a2= '';

+----+-------------+--------+------------+-------+---------------+-----------+---------+------+------+----------+--------------------------+
| id | select_type | table  | partitions | type  | possible_keys | key       | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+--------+------------+-------+---------------+-----------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | test02 | NULL       | index | idx_a1_a2     | idx_a1_a2 | 20      | NULL |    1 |   100.00 | Using where; Using index |
+----+-------------+--------+------------+-------+---------------+-----------+---------+------+------+----------+--------------------------+
1 row in set, 1 warning (0.00 sec)


```



- using where （需要回表查询）
假设age是索引列
但查询语句select age,name from ...where name =...，此语句中必须回原表查Name，因此会显示using where

```mysql
explain select a1,a3 from test02 where a3 = '' ; --a3需要回原表查询
```



-  impossible where ： where子句永远为false

```mysql
explain select * from test02 where a1='x' and a1='y';


+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra            |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Impossible WHERE |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------+
1 row in set, 1 warning (0.00 sec)

```



- Using join buffer

extra中的一个选项，作用，Mysql引擎使用了连接缓存

## 七、优化案例


### 7.1 SQL优化示例

```mysql
create table test03
(
  a1 int(4) not null,
  a2 int(4) not null,
  a3 int(4) not null,
  a4 int(4) not null
);
alter table test03 add index idx_a1_a2_a3_4(a1,a2,a3,a4) ;

explain select a1,a2,a3,a4 from test03 where a1=1 and a2=2 and a3=3 and a4 =4; --推荐写法，因为索引的使用顺序（where后面的顺序）和复合索引的顺序一致

explain select a1,a2,a3,a4 from test03 where a4=1 and a3=2 and a2=3 and a1 =4; --虽然编写的顺序和索引顺序不一致，但是sql在真正执行前经过了SQL优化器的调整，结果与上条SQL是一致的。
--以上 2个SQL，使用了全部的复合索引

explain select a1,a2,a3,a4 from test03 where a1=1 and a2=2 and a4=4 order by a3; 
--以上SQL用到了a1 a2两个索引，该两个字段 不需要回表查询using index;而a4因为跨列使用，造成了该索引失效，需要回表查询 因此是using where；虽然a4由于跨列导致索引失效，但是where(a1, a2) 和order by a3任然构成using index的条件，所以不会出现using filesort 以上可以通过 key_len进行验证

explain select a1,a2,a3,a4 from test03 where a1=1 and a4=4 order by a3; 
--以上SQL出现了 using filesort(文件内排序，“多了一次额外的查找/排序”) ：不要跨列使用( where和order by 拼起来，不要跨列使用)，这里a4索引失效，where(a1)和order by a3不构成using index的条件，所以出现using filesort


explain select a1,a2,a3,a4 from test03 where a1=1 and a4=4 order by a2 , a3;
--不会using filesort，由于 where 和order拼起来 a1, a4跨列失效不管，a2, a3 既where和order by 拼起来不跨列，所以不会有using filesort
```



下边我们了解一下单表优化、两表优化、三表优化

### 7.1 单表优化
```mysql
create table book(
    bid int(4) primary key,
    name varchar(20) not null,
    authorid int(4) not null,
    publicid int(4) not null,
    typeid int(4) not null 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

insert into book values(1,'tjava',1,1,2);
insert into book values(2,'python',2,1,2);
insert into book values(3,'sql',3,2,1);
insert into book values(4,'js',4,2,3);
```

查询authorid=1且 typeid为2或3的bid

```mysql
explain select bid from book where typeid in(2,3) and authorid=1 order by typeid desc;

+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
|  1 | SIMPLE      | book  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    4 |    25.00 | Using where; Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------------------+
1 row in set, 1 warning (0.00 sec)


优化：加索引
alter table book add index idx_bta (bid,typeid,authorid);

explain select bid from book where typeid in(2,3) and authorid=1 order by typeid desc;


+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+------------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                                    |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+------------------------------------------+
|  1 | SIMPLE      | book  | NULL       | index | NULL          | idx_bta | 12      | NULL |    4 |    25.00 | Using where; Using index; Using filesort |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+------------------------------------------+
1 row in set, 1 warning (0.00 sec)


索引一旦进行升级优化，需要将之前废弃的索引删掉，防止干扰。
drop index idx_bta on book;

根据SQL实际解析的顺序，调整索引的顺序：
alter table book add index idx_tab (typeid,authorid,bid); --虽然可以回表查询bid，但是将bid放到索引中 可以提升使用using index ;

再次优化（之前是index级别）：思路。因为in查询有时会变为范围查询，因此交换索引的顺序，将typeid in(2,3) 放到最后。
drop index idx_tab on book;
alter table book add index idx_atb (authorid,typeid,bid);
explain select bid from book where authorid=1 and typeid in(2,3) order by typeid desc;

+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | book  | NULL       | range | idx_atb       | idx_atb | 8       | NULL |    2 |   100.00 | Using where; Using index |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+--------------------------+
1 row in set, 1 warning (0.00 sec)

```


本例中同时出现了Using where（需要回原表）、Using index（不需要回原表）

原因：where  authorid=1 and  typeid in(2,3) 中 authorid在索引(authorid,typeid,bid)中，因此不需要回原表（直接在索引表中能查到），而typeid虽然也在索引(authorid,typeid,bid)中，但是含in的范围查询已经使该typeid索引失效，因此相当于没有typeid这个索引，所以需要回原表（using where）

还可以通过key_len证明in可以使索引失效。

例如以下没有了in，则不会出现using where

```mysql
 explain select bid from book where authorid=1 and typeid=3  order by typeid desc;
 
+----+-------------+-------+------------+------+---------------+---------+---------+-------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key     | key_len | ref         | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+---------+---------+-------------+------+----------+-------------+
|  1 | SIMPLE      | book  | NULL       | ref  | idx_atb       | idx_atb | 8       | const,const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+---------+---------+-------------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

```




**小结：**

a.最佳左前缀，保持索引的定义和使用的顺序一致性 

b.索引需要逐步优化  

c.将含in的范围查询放到where条件的最后，防止失效



### 7.2 两表优化

```mysql
create table teacher2(
    tid int(4) primary key,
    cid int(4) not null
);
insert into teacher2 values(1,2);
insert into teacher2 values(2,1);
insert into teacher2 values(3,3);

create table course2(
    cid int(4),
    cname varchar(20)
);
insert into course2 values(1,'java');
insert into course2 values(2,'python');
insert into course2 values(3,'kotlin');
```

左连接：

```mysql
explain select * from teacher2 t left outer join course2 c
on t.cid=c.cid where c.cname='java';
```

索引往哪张表加？

- 小表驱动大表 
- 索引建立经常使用的字段上 （本题 t.cid=c.cid可知，t.cid字段使用频繁，因此给该字段加索引） [一般情况对于左外连接，给左表加索引；右外连接，给右表加索引]

例如：

小表：10
大表：300
where 	小表.x 10 = 大表.y 300;  --循环了几次？10
		大表.y 300= 小表.x 10	 --循环了300次


```c
select ...where 小表.x10=大表.x300;
for(int i=0;i<小表.length10;i++)
{
	for(int j=0;j<大表.length300;j++)
	{
		...
	}
}
```


```c
select ...where 大表.x300=小表.x10 ;
for(int i=0;i<大表.length300;i++)
{
	for(int j=0;j<小表.length10;j++)
	{
		...
	}
}
```

**以上2个FOR循环，最终都会循环3000次；但是对于双层循环来说：一般建议将数据小的循环放外层；数据大的循环放内层。**

```mysql
--所以当编写 ..on t.cid=c.cid 时，将数据量小的表放左边（假设此时t表数据量小）
alter table teacher2 add index index_teacher2_cid(cid) ;
alter table course2 add index index_course2_cname(cname);


explain select * from teacher2 t left outer join course2 c on t.cid=c.cid where c.cname='java';
+----+-------------+-------+------------+------+---------------------+---------------------+---------+--------------------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys       | key                 | key_len | ref                | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+--------------------+------+----------+-------------+
|  1 | SIMPLE      | c     | NULL       | ref  | index_course2_cname | index_course2_cname | 63      | const              |    1 |   100.00 | Using where |
|  1 | SIMPLE      | t     | NULL       | ref  | index_teacher2_cid  | index_teacher2_cid  | 4       | sql_optimize.c.cid |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------------+---------------------+---------+--------------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)

```

**小结：**

a.小表驱动大表  

b.索引建立在经常查询的字段上



### 7.3 三张表优化A B C

多表优化一下原则同样适用

a.小表驱动大表  

b.索引建立在经常查询的字段上



**小结：**

i.如果 (a,b,c,d)复合索引  和使用的顺序全部一致(且不跨列使用)，则复合索引全部使用。如果部分一致(且不跨列使用)，则使用部分索引。
	select a,c where  a = and b= and d= 

ii.where和order by 拼起来，不要跨列使用 


iii.using temporary：需要额外再多使用一张表， 一般出现在group by语句中，已经有表了，但不适用，必须再来一张表。

解析过程：			
from .. on.. join ..where ..group by ....having ...select dinstinct ..order by limit ...

```mysql
explain select * from test03 where a2=2 and a4=4 group by a2,a4 ;--没有using temporary，因为group by字段全在where中
explain select * from test03 where a2=2 and a4=4 group by a3 ;

以上两行sql均会出现如下错误：
mysql> explain select a4 from test03 where a2=2 and a3=4 group by a2 ;
ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'sql_optimize.test03.a4' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

原因：select 和 group by用的字段不一样，如果group by a2, 那么也要select a2，如果使用group by而select中又不适用聚合函数，那么查询无意义



```


## 八、避免索引失效的一些原则 

（1）复合索引
- 	a.复合索引，不要跨列或无序使用（最佳左前缀）
-   b.复合索引，尽量使用全索引匹配

（2）不要在索引上进行任何操作（计算、函数、类型转换），否则索引失效

```mysql
select ..where A.x = .. ;  --假设A.x是索引
不要：select ..where A.x*3 = .. ;
explain select * from book where authorid = 1 and typeid = 2;--用到了2个索引
explain select * from book where authorid = 1 and typeid*2 = 2;--用到了1个索引
explain select * from book where authorid*2 = 1 and typeid*2 = 2;--用到了0个索引
explain select * from book where authorid*2 = 1 and typeid = 2;--用到了0个索引,原因：对于复合
索引，如果左边失效，右侧全部失效。(a,b,c)，例如如果 b失效，则b c同时失效。
```

（3）复合索引不能使用不等于（!=  <>）或is null (is not null)，否则自身以及右侧所有全部失效
复合索引中如果有>，则自身和右侧索引全部失效。

```mysql
explain select * from book where authorid = 1 and typeid =2; --使用索引 type=ref
explain select * from book where authorid != 1 and typeid =2; --不使用索引 type=all
explain select * from book where authorid != 1 and typeid !=2;--不使用索引 type=all
```

**注意：SQL优化，是一种概率层面的优化。至于是否实际使用了我们的优化，需要通过explain进行推测体验概率情况(< > =)：原因是服务层中有SQL优化器，可能会影响我们的优化**

```mysql
drop index idx_typeid on book;
drop index idx_authroid on book;
alter table book add index idx_book_at (authorid,typeid);
explain select * from book where authorid = 1 and typeid =2;--复合索引全部使用
explain select * from book where authorid > 1 and typeid =2;--复合索引中如果有>，则自身和右侧索引全部失效。
explain select * from book where authorid = 1 and typeid >2;--复合索引全部使用
----明显的概率问题---
explain select * from book where authorid < 1 and typeid =2;--复合索引只用到了1个索引
explain select * from book where authorid < 4 and typeid =2;--复合索引全部失效
```

我们学习索引优化 ，是一个大部分情况适用的结论，但由于SQL优化器等原因该结论不是100%正确。一般而言， 范围查询（> <  in），之后的索引失效。

（4）补救。尽量使用索引覆盖（using index）（a,b,c）

```mysql
select a,b,c from xx..where a=  .. and b =.. ;
```


（5）like尽量以“常量”开头，不要以'%'开头，否则索引失效

```mysql
select * from xx where name like '%x%'; --name索引失效
explain select * from teacher  where tname like '%x%'; --tname索引失效
explain select * from teacher  where tname like 'x%';
explain select tname from teacher  where tname like '%x%'; --如果必须使用like '%x%'进行模糊查询，可以使用索引覆盖 挽救一部分。
```



（6）尽量不要使用类型转换（显示、隐式），否则索引失效


```mysql
explain select * from teacher where tname = 'abc' ;
explain select * from teacher where tname = 123 ;//程序底层将 123 -> '123'，即进行了类型转换，因此索引失效
```



（7）尽量不要使用or，否则索引失效

```mysql
explain select * from teacher where tname ='' or tcid >1 ; --将or左侧的tname 失效。
```



（8）一些其他的优化方法

- exist和in

```mysql
select ..from table where exist (子查询) ;
select ..from table where 字段 in  (子查询) ;
如果主查询的数据集大，则使用In,效率高。
如果子查询的数据集大，则使用exist,效率高。	

exist语法： 将主查询的结果，放到子查需结果中进行条件校验（看子查询是否有数据，如果有数据 则校验成功）  ，
	    如果 复合校验，则保留数据；

select tname from teacher where exists (select * from teacher) ; 
--等价于select tname from teacher
```

- order by 优化
```
	using filesort 有两种算法：双路排序、单路排序 （根据IO的次数）
	MySQL4.1之前默认使用双路排序
	双路：扫描2次磁盘
		1：从磁盘读取排序字段,对排序字段进行排序（在buffer中进行的排序）  
		2：扫描其他字段
	
	IO较消耗性能
	MySQL4.1之后默认使用单路排序
		只读取一次（全部字段），在buffer中进行排序。但单路排序会有一定的隐患 （不一定真的是“单路|1次IO”，有可能多次IO）。原因：如果数据量特别大，则无法将所有字段的数据一次性读取完毕，因此会进行“分片读取、多次读取”。
	
	注意：单路排序比双路排序会占用更多的buffer。
	
	单路排序在使用时，如果数据大可以考虑调大buffer的容量大小：set max_length_for_sort_data = 1024 单位byte
	
	如果max_length_for_sort_data值太低，则mysql会自动从 单路->双路 太低：需要排序的列的总大小超过了max_length_for_sort_data定义的字节数）
```

**提高order by查询的策略**
- a.选择使用单路、双路 ，调整buffer的容量大小；
- b.避免select * ... 
- c.复合索引不要跨列使用 ，避免using filesort
- d.保证全部的排序字段 排序的一致性（都是升序 或 降序）	

## 九、使用SQL日志排查慢SQL

慢查询SQL日志：MySQL提供的一种日志记录，用于记录MySQL种响应时间超过阀值的SQL语（long_query_time，默认10秒）
慢查询日志默认是关闭的，建议：开发调优是打开，而最终部署时关闭。

检查是否开启了慢查询日志

```mysql
show variables like '%slow_query_log%';
```

临时开启

```mysql
set global slow_query_log = 1;  --在内存中开启
```

永久开启

```shell
/etc/my.cnf 中追加配置：
vi /etc/my.cnf 
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/localhost-slow.log
```

慢查询阀值

```mysql
show variables like '%long_query_time%';
```


临时设置阀值

```mysql
exit--设置完毕后，重新登陆后起效 （不需要重启服务）
```

永久设置阀值：		

```shell
/etc/my.cnf 中追加配置：
vi /etc/my.cnf 
[mysqld]
long_query_time=3
```

查询超过阀值的SQL


```mysql
select sleep(4);
select sleep(5);
select sleep(3);
select sleep(3);
--查询超过阀值的SQL：
show global status like '%slow_queries%';
```

(1)慢查询的sql被记录在了日志中，因此可以通过日志 查看具体的慢SQL

```mysql
cat /var/lib/mysql/localhost-slow.log
```

(2)通过mysqldumpslow工具查看慢SQL，可以通过一些过滤条件 快速查找出需要定位的慢SQL

```mysql
mysqldumpslow --help
s：排序方式
r:逆序
l:锁定时间
g:正则匹配模式

--获取返回记录最多的3个SQL
	mysqldumpslow -s r -t 3  /var/lib/mysql/localhost-slow.log

--获取访问次数最多的3个SQL
	mysqldumpslow -s c -t 3 /var/lib/mysql/localhost-slow.log

--按照时间排序，前10条包含left join查询语句的SQL
	mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/localhost-slow.log

语法：
	mysqldumpslow 各种参数  慢查询日志的文件
```


(3)全局查询日志 ：记录开启之后的全部SQL语句。 （这次全局的记录操作仅仅在调优、开发过程中打开即可，在最终的部署实施时 一定关闭）

```mysql
show variables like '%general_log%';
		
--执行的所有SQL记录在表中
set global general_log = 1 ;--开启全局日志
set global log_output='table' ; --设置 将全部的SQL 记录在表中
	
--执行的所有SQL记录在文件中
set global log_output='file' ;
set global general_log = on ;
set global general_log_file='/tmp/general.log';

开启后，会记录所有SQL：会被记录 mysql.general_log表中。
select * from  mysql.general_log ;
```



## 十一、锁机制 ：解决因资源共享而造成的并发问题

示例：买最后一件衣服X
A:  	X	买 ：  X加锁 ->试衣服...下单..付款..打包 ->X解锁
B:	X       买：发现X已被加锁，等待X解锁，   X已售空



**分类：**
操作类型：

- 读锁（共享锁）： 对同一个数据（衣服），多个读操作可以同时进行，互不干扰。
- 写锁（互斥锁）： 如果当前写操作没有完毕（买衣服的一系列操作），则无法进行其他的读操作、写操作

操作范围：

- 表锁 ：一次性对一张表整体加锁。如MyISAM存储引擎使用表锁，开销小、加锁快；无死锁；但锁的范围大，容易发生锁冲突、并发度低。
- 行锁 ：一次性对一条数据加锁。如InnoDB存储引擎使用行锁，开销大，加锁慢；容易出现死锁；锁的范围较小，不易发生锁冲突，并发度高（很小概率 发生高并发问题：脏读、幻读、不可重复读、丢失更新等问题）。
- 页锁		

### 11.2 InnoDB行锁

本节内容可以先参考：[MySQL共享锁与排他锁](https://blog.csdn.net/u014292162/article/details/83271299)

行表（InnoDB）

```mysql
create table linelock(
	id int(5) primary key auto_increment,
	name varchar(20)
)engine=innodb;

insert into linelock(name) values('1');
insert into linelock(name) values('2');
insert into linelock(name) values('3');
insert into linelock(name) values('4');
insert into linelock(name) values('5');
```


mysql默认自动commit；

为了研究行锁，暂时将自动commit关闭;  set autocommit =0 ; 以后sql需要通过commit提交执行

```mysql
会话0： 写操作
	insert into linelock(name) values('6');
	
会话1： 写操作 同样的数据
	update linelock set name='ax' where id = 6;
	
对行锁情况：
	1.如果会话x对某条数据a进行DML操作（研究时：关闭了自动commit的情况下），则其他会话必须等待会话x结束事务(commit/rollback)后才能对数据a进行操作。
	2.表锁是通过unlock tables，也可以通过事务解锁; 行锁是通过事务解锁。

行锁，操作不同数据：
会话0： 写操作
	insert into linelock values(8,'a8') ;
会话1： 写操作，不同的数据
	update linelock set name='ax' where id = 5;
	行锁，一次锁一行数据；因此如果操作的是不同数据，则不干扰。
```

**行锁的注意事项：**

a.如果没有索引，则行锁会转为表锁

```mysql
--给name字段加一个索引
show index from linelock;
alter table linelock add index idx_linelock_name(name);

会话0： 写操作
	update linelock set name = 'ai' where name = '3';
会话1： 写操作， 不同的数据
	update linelock set name = 'aiX' where name = '4';

会话0： 写操作
	update linelock set name = 'ai' where name = 3;
		
会话1： 写操作， 不同的数据
	update linelock set name = 'aiX' where name = 4;
		
--可以发现，数据被阻塞了（加锁）
-- 原因：如果索引类发生了类型转换，则索引失效。 因此此次操作，会从行锁转为表锁。
```


b.行锁的一种特殊情况：间隙锁：值在范围内，但却不存在
```mysql
mysql> select * from linelock;
+----+------+
| id | name |
+----+------+
|  1 | 1    |
|  2 | 2    |
|  3 | 3    |
|  4 | 4    |
|  6 | 6    |
|  8 | a8   |
|  5 | ax   |
+----+------+
--此时linelock表中 没有id=7的数据
	 update linelock set name ='x' where id >1 and id<9;  --即在此where范围中，没有id=7的数据，则id=7的数据成为间隙。
	 
	  insert into linelock values(7,'a7');
间隙：Mysql会自动给 间隙 加索 ->间隙锁。即本题会自动给id=7的数据加间隙锁（行锁）。
行锁：如果有where，则实际加索的范围就是where后面的范围（不是实际的值）
```



**行锁：**
InnoDB默认采用行锁；
缺点： 比表锁性能损耗大。
优点：并发能力强，效率高。
因此建议，高并发用InnoDB，否则用MyISAM。	