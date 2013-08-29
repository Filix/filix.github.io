---
layout: post
title: PHP SESSION -- configuration
tag: [php]
keywords: 子非吾,php,session,技术博客,it
---
#PHP SESSION -- configuration
Session(又称会话)作为http协议无状态的一种解决方案，在网站建设中被广泛使用。

关于PHP中SESSION的内部执行流程，同学们可以参阅这 [http://www.360weboy.com/php/session-mechanism/](http://www.360weboy.com/php/session-mechanism/)，所以本篇文章不在介绍为什么使用SESSION、SESSION和COOKIE有什么区别之类的问题。

这篇文章主要介绍PHP中关于SESSION的一些配置和PHP为我们提供的可以在程序中修改这写参数的函数。

PHP中关于SESSION的配置大约有30来个，下面我们依次对他们介绍。

###session.save_handler
session.save_handler定义了来存储和获取与SESSION关联的数据的处理器的名字,简单的说就是用什么存储SESSION信息，文件还是Mysql或者memcache。

默认值：files。所以这就是我们开始学习SESSION时常听说的:SESSION就是保存在服务器上的一些文件。

可以通过session_set_save_handler()函数覆盖该配置。

###session.save_path
session.save_path定义了存储SESSION的位置，如果使用files，则该参数就是存放SESSION文件的目录；如果使MySql或者memcache，那么就是这个服务器的host和端口号等信息。

默认值： /tmp。

可以通过使用session_save_path()函数覆盖该配置。

此外，该参数还有一个可选的整数N参数来决定会话文件的分布深度。因为当一个站点的用户很多的时候，一个目录下的会话文件也会很多，所以我们可以把这些文件放在不同的子目录里。例如： 设置"4;/tmp/sessions"会将会话文件存在/tmp/sessions/a/b/c/d/sess_abcd123456789的位置。当使用N参数是要实现创建好这些子目录，因为PHP不会自动去创建这些目录。在ext/session目录下有个小的shell脚本mod_files.sh可以用来做这件事。注意当N大于0时，PHP将不会启动执行垃圾回收机制。此外，设置N参数时必须要用双引号括起来，因为分隔符(;)在php.ini中也是注释符。

###session_name
session.name指定会话用以做cookie的名字，默认值：PHPSESSIONID。如果你经常研究一些网站的cookie，会经常看到这样的一个cookie，就是因为这个设置的原因。

也可以通过session_name()函数指定。当php.ini中设置session.auto_start为true时，该函数不起作用。这是因为SESSION已经被启动了，因为你不能再去修改他的名字了。所以该函数也必须要在session_start()函数之前被调用。

###session.auto_start
session.auto_start指定会话模块是不是在一个请求开始时自动启动一个会话。

默认为：0。

可以使用session_start()函数手动启动一个会话。

不启动session就使用SESSION是没效果的（好像是句废话），很多初学者确实经常被这点困扰过。

###session.serialize_handler
session.serialize_handler定义了序列化和反序列化会话的处理器名字。

默认值： php

###session.gc_probability 和 session.gc_divisor
session.gc_probability和session.gc_divisor一起使用，定义了gc（垃圾回收）进程启动的频率。
此频率用 session.gc_probability/session.gc_divisor 计算得来。列如1/100意味着每个请求有1%的可能启动GC进程。

session.gc_probability 默认值： 1

session.gc_divisor 默认值： 100

###session.gc_maxlifetime
session.gc_maxlifetime 指定了SESSION的过期时间。

如果使用默认的基于文件的会话处理器，则文件系统必须保持跟踪访问时间（atime）。Windows FAT 文件系统不行，因此如果必须使用 FAT 文件系统或者其他不能跟踪 atime 的文件系统，那就不得不想别的办法来处理会话数据的垃圾回收。自 PHP 4.2.3 起用 mtime（修改时间）来代替了 atime。因此对于不能跟踪 atime 的文件系统也没问题了。






