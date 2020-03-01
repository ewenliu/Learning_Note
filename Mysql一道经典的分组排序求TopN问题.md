#### 问题

找出每个班成绩排名前N名的同学

#### 环境

OS: Win10

DB: 5.7.25 MySQL Community Server

#### 建立测试学生表

表结构字段由 **id，name，score，classid**组成


```c
CREATE TABLE student (
id int(11) AUTO_INCREMENT,
name varchar(50),
score int(11),
classid int(11),
primary key(id)
);

mysql> desc student;
+---------+-------------+------+-----+---------+----------------+
| Field   | Type        | Null | Key | Default | Extra          |
+---------+-------------+------+-----+---------+----------------+
| id      | int(11)     | NO   | PRI | NULL    | auto_increment |
| name    | varchar(50) | YES  |     | NULL    |                |
| score   | int(11)     | YES  |     | NULL    |                |
| classid | int(11)     | YES  |     | NULL    |                |
+---------+-------------+------+-----+---------+----------------+
4 rows in set (0.02 sec)


```

#### 插入测试数据

```c
insert into student (name, score, classid) values("hhr", 70, 1);
insert into student (name, score, classid) values("wsx", 99, 1);
insert into student (name, score, classid) values("wk", 62, 1);
insert into student (name, score, classid) values("zxf", 88, 2);
insert into student (name, score, classid) values("lyw", 100, 2);

mysql> select * from student;
+----+------+-------+---------+
| id | name | score | classid |
+----+------+-------+---------+
|  1 | hhr  |    70 |       1 |
|  2 | wsx  |    99 |       1 |
|  3 | wk   |    62 |       1 |
|  4 | zxf  |    88 |       2 |
|  5 | lyw  |   100 |       2 |
+----+------+-------+---------+
5 rows in set (0.00 sec)
```

#### 查询每个班的最高分数

```c
mysql> select max(score) as max_score, classid from student group by classid;
+-----------+---------+
| max_score | classid |
+-----------+---------+
|        99 |       1 |
|       100 |       2 |
+-----------+---------+
2 rows in set (0.02 sec)
```
可以发现仅能查出每门课的最高分，但是最高分属于谁查不到

如果想直接查学生姓名，即select中再加入name字段的话，sql会报错如下

```c
mysql> select max(score) as max_score, name, classid from student group by classid;
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'store.student.name' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

这是因为和group by配合使用的select中只能出现聚合函数或者group by中本身使用的字段

> 如果不使用max，min，count等聚合函数，那么没有必要用group by来做分组，group by的意义就是用来求分组后每组的最大值/最小值/平均值/累加等

如果想查询最高分学生姓名，那么使用子查询即可
```c
 select * from student where score in (select max(score) as max_score from student group by classid);
 
 +----+------+-------+---------+
| id | name | score | classid |
+----+------+-------+---------+
|  2 | wsx  |    99 |       1 |
|  5 | lyw  |   100 |       2 |
|  6 | ly   |   100 |       2 |
+----+------+-------+---------+
3 rows in set (0.00 sec)

需要注意的是，如果主查询数据集比子查询大，用in，反之用exists
```

#### 怎么取得每个班的成绩前几名呢

> 只会查第一名是不够的。

虽然最高的第一名查出来了，但是实际运用场景中比如我们要找前3名，前10名是，怎么查？group by的max函数只能取最大值，怎么玩？？？

不多说，直接上代码

先按照课程关系生成一张连接表（笛卡尔积），这里用的是**左外连接**，如果不懂连接查询的话可以先百度搜一下什么是sql连接查询，sql连接查询的分类等等
```c
select * from student a left join student b on a.classid=b.classid;

# 查询结果：
+----+------+-------+---------+------+------+-------+---------+
| id | name | score | classid | id   | name | score | classid |
+----+------+-------+---------+------+------+-------+---------+
|  1 | hhr  |    70 |       1 |    1 | hhr  |    70 |       1 |
|  2 | wsx  |    99 |       1 |    1 | hhr  |    70 |       1 |
|  3 | wk   |    62 |       1 |    1 | hhr  |    70 |       1 |
|  1 | hhr  |    70 |       1 |    2 | wsx  |    99 |       1 |
|  2 | wsx  |    99 |       1 |    2 | wsx  |    99 |       1 |
|  3 | wk   |    62 |       1 |    2 | wsx  |    99 |       1 |
|  1 | hhr  |    70 |       1 |    3 | wk   |    62 |       1 |
|  2 | wsx  |    99 |       1 |    3 | wk   |    62 |       1 |
|  3 | wk   |    62 |       1 |    3 | wk   |    62 |       1 |
|  4 | zxf  |    88 |       2 |    4 | zxf  |    88 |       2 |
|  5 | lyw  |   100 |       2 |    4 | zxf  |    88 |       2 |
|  4 | zxf  |    88 |       2 |    5 | lyw  |   100 |       2 |
|  5 | lyw  |   100 |       2 |    5 | lyw  |   100 |       2 |
+----+------+-------+---------+------+------+-------+---------+
13 rows in set (0.00 sec)
```

加入限制条件a.score < b.score用来筛选，这里由于是左外连接，所以当a.score不小于b.score时，b表相关字段查出来为**NULL**

```c
select * from student a left join student b on a.classid=b.classid and a.score<b.score;

# 查询结果：
+----+------+-------+---------+------+------+-------+---------+
| id | name | score | classid | id   | name | score | classid |
+----+------+-------+---------+------+------+-------+---------+
|  3 | wk   |    62 |       1 |    1 | hhr  |    70 |       1 |
|  1 | hhr  |    70 |       1 |    2 | wsx  |    99 |       1 |
|  3 | wk   |    62 |       1 |    2 | wsx  |    99 |       1 |
|  4 | zxf  |    88 |       2 |    5 | lyw  |   100 |       2 |
|  2 | wsx  |    99 |       1 | NULL | NULL |  NULL |    NULL |
|  5 | lyw  |   100 |       2 | NULL | NULL |  NULL |    NULL |
+----+------+-------+---------+------+------+-------+---------+
6 rows in set (0.00 sec)

```

按照(a.classid,a.name,a.score) 作为**多字段分组**条件（如果不明白什么是多字段分组可以百度），查出同一列中a成绩比b成绩小的行数

从上面查询结果来看，当classid为1时，wk的成绩比hhr和wsx都小，所以下面count(b.id)这一列查出来为2，即代表wk的classid =1的成绩要小于2个同学;

从上面表来看，当classid为1时，hhr的成绩仅比wsx小，所以下面count(b.id)这一列查出来为1，代表hhr的classid =1的成绩要小于1个同学;

从上面表来看，当classid为1时，wsx的成绩不比任何人小（即最大），所以下面count(b.id)这一列查出来为0;

同理可以得出当classid为2时的count(b.id)情况

```c
select a.classid,a.name,a.score ,count(b.id)from student a left join student b 
on a.classid=b.classid and a.score<b.score 
GROUP BY a.classid,a.name,a.score ;
+---------+------+-------+-------------+
| classid | name | score | count(b.id) |
+---------+------+-------+-------------+
|       1 | hhr  |    70 |           1 |
|       1 | wk   |    62 |           2 |
|       1 | wsx  |    99 |           0 |
|       2 | lyw  |   100 |           0 |
|       2 | zxf  |    88 |           1 |
+---------+------+-------+-------------+
```


```c

既然上面查询结果中列count(b.id)说明的是**分数排名情况**，那么加入having count(b.id)<N 条件来查询排名前N的同学

比如having count(b.id)<2，即查询的是count(b.id)等于0，1的情况，就是筛选课程分数不比任何人小（0）和人数只比1个人小的列

```

```c
# 查询各科排名前2的学生

select a.classid,a.name,a.score ,count(b.id)from student a left join student b 
on a.classid=b.classid and a.score<b.score 
GROUP BY a.classid,a.name,a.score 
having count(b.id)<2;

+---------+------+-------+-------------+
| classid | name | score | count(b.id) |
+---------+------+-------+-------------+
|       1 | hhr  |    70 |           1 |
|       1 | wsx  |    99 |           0 |
|       2 | lyw  |   100 |           0 |
|       2 | zxf  |    88 |           1 |
+---------+------+-------+-------------+
4 rows in set (0.00 sec)

# 再把查询结果弄好看点，加一个排序条件order by a.classid, a.score，即先按照课程id排序，如果课程id相等，则按照分数排序，从高到低用desc

select a.classid,a.name,a.score from student a left join student b 
on a.classid=b.classid and a.score<b.score 
GROUP BY a.classid,a.name,a.score 
having count(b.id)<2 
order by a.classid,a.score desc;

+---------+------+-------+-------------+
| classid | name | score | count(b.id) |
+---------+------+-------+-------------+
|       1 | wsx  |    99 |           0 |
|       1 | hhr  |    70 |           1 |
|       2 | lyw  |   100 |           0 |
|       2 | zxf  |    88 |           1 |
+---------+------+-------+-------------+
4 rows in set (0.00 sec)



# 最终sql

select 
	a.classid,a.name,a.score
from 
	student a 
left join 
	student b 
on 
	a.classid=b.classid and a.score<b.score
group by 
	a.classid,a.name,a.score 
having 
	count(b.id)<2
order by 
	a.classid,a.score desc;

5 rows in set (0.01 sec)
```

总的来说，有三个难以理解的地方

1. 为什么使用连接查询
2. 为什么使用a.score<b.score
3. 为什么使用count(b.id)来反应排名情况？

如果以上3个问题想的明白，分组排序TopN也就清楚了。

