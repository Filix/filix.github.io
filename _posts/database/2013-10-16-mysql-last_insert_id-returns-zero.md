---
layout: post
title: 记一次MySql之last_insert_id()返回0排查
tag: [database]
keywords: 子非吾,MySql,last_insert_id,return 0,技术博客,it
---
#记一次MySql之last_insert_id()返回0排查

上周有一次在MySql中执行了插入语句之后，无意调用了一下`select last_insert_id()`结果返回0，实际上插入语句成功执行了，但是该函数却始终返回0，百思不得其解。

各种google之后，最后，终于发现了问题，写出来和大家分享下。

创建测试表：

	create table test(
		id int not null auto_increment primary key,
		name varchar(20) not null
	)
	
	
然后执行：

	insert into test (id, name) values (1,'aaa');
	select last_insert_id();
	
结果如下：

	mysql> select last_insert_id();
	+------------------+
	| last_insert_id() |
	+------------------+
	|                0 |
	+------------------+
	1 row in set (0.00 sec)
	
但是我们插入的数据却是实实在在的已经插入成功了：

	mysql> select * from test;
	+----+------+
	| id | name |
	+----+------+
	|  1 | aaa  |
	+----+------+
	1 row in set (0.00 sec)
	
原因是我们制定了主键的值，也就是id字段的值。如果我们，不指定这个值再试试：

	mysql> insert into test (name) values('bbb');
	Query OK, 1 row affected (0.00 sec)
	
	mysql> select last_insert_id();
	+------------------+
	| last_insert_id() |
	+------------------+
	|                2 |
	+------------------+
	1 row in set (0.00 sec)
	

问题虽然找到了，我们继续深入研究。

#####批量插入

如果使用批量插入，`last_insert_id()`返回第一条插入的记录的id：

	mysql> insert into test (name) values('ccc'),('ddd');
	Query OK, 2 rows affected (0.00 sec)
	Records: 2  Duplicates: 0  Warnings: 0

	mysql> select last_insert_id();
	+------------------+
	| last_insert_id() |
	+------------------+
	|                3 |
	+------------------+
	1 row in set (0.00 sec)
	
现在一共有4条记录，这条语句返回的是第三条数据的id。

#####使用phpmyadmin返回0
如果你使用的是phpmyadmin里面的命令行模式，如果按照正确方式插入，缺还是返回0，那么有可能是因为phpmyadmin配置文件中的`PersistentConnections`参数被设置成false了。这种情况下每次查询都会新建一次连接，所以会返回0。

就是这么多了，希望对你有帮助。
