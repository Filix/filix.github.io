---
layout: post
title: MySql之on duplicate key update详解
tag: [database]
keywords: 子非吾,MySql,on duplicate key update,技术博客,it
---
#MySql之on duplicate key update详解
在我们的日常开发中，你是否遇到过这种情景：查看某条记录是否存在，不存在的话创建一条新记录，存在的话更新某些字段。你的处理方式是不是就是按照下面这样？

	$result = mysql_query('select * from xxx where id = 1');
	$row = mysql_fetch_assoc($result);
	if($row){
		mysql_query('update ...');
	}else{
		mysql_query('insert ...');
	}
	
这样的写法可能有如下几点缺陷：


 1. 麻烦。一个很简单的逻辑缺要写十来行代码。    
 2. 少量的性能消耗。执行了两次sql，按照一条sql一去一回两次网络传输的话，那么这就是4次。    
 3. 在高并发下会出问题。比如当我们获取到了需要的数据，在更新之前，有另外一个请求恰好删除了该条记录，我们的更新操作就没起到作用；再或者，如果我们更新操作的写法有问题，比如更新列a，我们使用`a = $row[a] + 1`而不是`a = a +1`这种原子性的操作，有可能别的请求已经修改过了该字段，从而造成数据出错。


幸好，MySql考虑到了这点，提供了`insert … on duplicate key update`的语法，该语法在insert的时候，如果insert的数据会引起唯一索引（包括主键索引）的冲突，即这个唯一值重复了，则不会执行insert操作，而执行后面的update操作。

例如，现在有表test，test表中有字段a，在a上有主键或者唯一索引，并且表中只有一条a=1, b=1的数据，现在执行如下的sql：

	insert into test (a,b) values (1,2) on duplicate key update b = b = 1;
	#因为a=1的记录已存在了，所以不会执行insert，而会在该条记录上执行update语言`b=b+1`,记录会变成a=1,b=2
	insert into test (a,b) values (2,2) on duplicate key update b = b + 1;
	#a=2的记录不存在，所以执行insert
	
这样我们就无需在应用程序里面再去判断记录是否存在了，也无需关系高并发下数据出错的情况了。
	
如果行作为新记录被插入，则受影响的行为1；如果原有记录被更新，则受影响行为2；如果原有记录已存在，但是更新的值和原有值相同，则受影响行为0。

#####多唯一索引冲突

为了测试方便，我们建了下面的数据表：

	create table test(
		a int not null primary key,
		b int not null UNIQUE key,
		c int not null
	)	
	
为了测试两个唯一索引都冲突的情况，然后插入下面的数据：
	
	insert into test values(1,1,1), (2,2,2);
	
然后执行：
	
	insert into test values(1,2,3) on duplicate key update c = c + 1;

因为a和b都是唯一索引，插入的数据在两条记录上产生了冲突，然而执行后只有第一条记录被修改：

	mysql> select * from test;
	+---+---+---+
	| a | b | c |
	+---+---+---+
	| 1 | 1 | 2 |
	| 2 | 2 | 2 |
	+---+---+---+
	2 rows in set (0.00 sec)
	
上面的语句等同于：

	update test set c=c+1 where a=1 or b = 2 limit 1;
	
如果`a=1 or b =2`匹配多条记录，只有第一条记录被更新。所以，一般情况下，我们应该避免在有多个唯一索引的表中使用`on duplicate key update`。

#####使用values()方法 
	
在update中可以使用`values()`方法引用在insert中的值，如：

	insert into test values(1,3,5) on duplicate key update c = values( c )+ 1;
	 
该语句会使a=1的记录中c字段的值更新为6，因为values(c)的值是引用的insert部分的值，在这个例子中就是`insert into test values(1,3,5) `中的5，所以最终更新的值为6。

#####last_insert_id()
如果表含有auto_increment字段，使用`insert … on duplicate key update`插入或更新后，`last_insert_id()`返回auto_increment字段的值。

#####并发控制
在使用例如MyISAM这样的表级锁的分区表上使用`insert … on duplicate key update`时，会锁住所有分区表，而在例如使用InnoDB这样的行级锁的分区表上则不会锁住所有分区表。


#####delayed选项
delayed选项会被忽略，当我们使用on duplicate key update时。





