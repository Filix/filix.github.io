---
layout: post
title: MySql存储引擎
tag: [database]
keywords: 子非吾,MySql存储引擎,技术博客,it
---
#MySql存储引擎

存储引擎(storage engine)是什么？本人自己的理解：存储引擎就是数据库底层定义的一套如何存放数据和索引（放在内存还是磁盘文件？在内容中按什么方法存？存文件怎么存？每个表一个文件还是共享一个文件？表数据和索引需要分开存吗？存在文件中按什么格式存？）、采用何种索引技巧（哈希还是b-tree？）、锁定策略以及如何向上跟用户提供各种接口等一系列问题答案的一套规则。

本文以MySql数据库为例，研究mysql有哪些存储引擎，以及各种存储引擎的应用场景。Mysql支持插件式的存储引擎，也就是完全可以将一个新的存储引擎加载到一个正在运行中的MySql中，而不影响MySql的运行。MySql支持的存储引擎只要有MyISAM，InnoDB，Memory，Archive，Merge，CSV，DNB Cluster， Maria，Falcon等，下面将分别介绍这些引擎。

###MyISAM存储引擎

MySql的默认存储引擎，支持较高的插入和查询速度，但是不支持事务和外键约束。

说MyISAM具有较高的插入和查询速度（跟InnoDB比较，因为这是最常见和最常使用的两种引擎，InnoDB在下面介绍），首先分析写的速度为什么比InnoDB快。InnoDB支持事务和完整性约束，为了保证事务的安全性InnoDB在写数据的同时要写日志，以防server crash掉的时候能从错误中恢复出来。另外，既然支持数据完整性，那么在插入数据的时候比如需要查询数据完整性，所以可以认为MyISAM快就快在没有这两步操作。至于为什么查询也快，请移步》MyISAM和InnoDB的查询比较

每个MyISAM的表有且只有三个与之相关的文件，分别是：表名.frm、表名.MYD、表名.MYI。.frm文件是每种存储引擎的表都不会缺少的文件，因为它定义了这个表的结构。.MYD文件存放数据表的数据。.MYI文件存放数据表的所有索引，不管这个表有几个索引，都存在这里。

说到数据文件，继续深入的研究下，数据究竟是如何存在数据文件中的。MyISAM支持3种存储格式：静态固定长度（Fixed），动态可变长度(Dynamic)和压缩（Compressed）。当然三种格式中 是否压缩是完全可以任由我们自己选择的，可以在创建表的时候通过 ROW_FORMAT 来指定 {COMPRESSED | DEFAULT}，也可以通过myisampack工具来进行压缩，默认是不压缩的。在非压缩的情况下，是静态和是动态，默认是有表字段的定义决定的，如果表中含有可变长度的类型（varchar，blob，text），那么默认就是动态可变长度的，相反则是静态固定长度的。当然，我们可以通过alter table table_name row_format fixed|dynamic去自由转换这两种方式，转换的结果是，varchar变为char（这里说的转换的结果是指的存储方式，并不是字段的类型变化了，desc table_name看到的还是varchar）。注意当表里有blob或text字段时是不能做这种转换的，因为这两种类型没有最大长度限制或者说他们的最大长度大于char类型的最大长度，所以不能转换。MyISAM的三种存储格式中，静态格式就最简单也是最安全的（至少对于崩溃而言）。静态格式也是最快的on-disk格式。快速来自于数据文件中的行在磁盘上被找到的容易方式：当按照索引中的行号查找一个行时，用行长度乘以行号。同样，当扫描一个表的 时候，很容易用每个磁盘读操作读一定数量的记录。对于动态存储更为复杂一点，因为每行有一个表明行有多长的头。当一个记录因为更新的结果被变得更长，该记录也可以在超过一个位置处结束。可以使用OPTIMIZE TABLE或myisamchk来对一个表整理碎片。如果在一个表中有你频繁访问或改变的固定长度列，表中也有一些可变长度列，仅为避免碎片而把这些可变长度列移到其它表可能是一个好主意。

MyISAM支持三种索引：B-tree，R-tree和Full-text索引。MySql中只有MyISAM引擎支持全文索引。

MyISAM引擎其余的特性请移步》  [MySql.com](MySql.com)


###InnoDB存储引擎
敬请期待