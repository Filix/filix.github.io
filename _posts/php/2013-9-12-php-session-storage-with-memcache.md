---
layout: post
title: PHP SESSION -- Storage With Memcache
tag: [php]
keywords: 子非吾,php,session,storage,memcache
---
#PHP SESSION -- Storage With Memcache
之前我们已经了解了php关于session方面的一些设置[PHP SESSION – configuration](/2013/08/29/php-session-configuration.html)，今天我们关注更高级和实用的东西--在php中使用memcache存储session。

随着用户的不断增加，在默认情况下（files），session文件数会越来越多，这么多的小文件放在一个目录里使查询这些文件变的越来越慢，虽然php提供了我们分级存放session文件的功能，这样虽然可以缓解文件过多造成的查询过慢的问题，但是仍然会有其他方面的一些缺点。比如，当我们有多台web服务器时，session同步的问题就出现了，这个问题我们又可以有多种方式解决，比如：共享session目录的方式、使用缓存服务器或数据库存储。    
所以这篇文章就主要介绍如何使用memcache存储session。

####准备工作
1. 安装memcache缓存服务器
2. 安装memcache扩展，并启用。php有两个memcache扩展，分表是memcache.so和memcached.so，分别对应着Memcache类和Memcached类，这是两个不同的扩展，初学者容易搞混。

###使用memcache类存储session

第一步，修改php的session处理器，使之从默认的文件变为memcache。    
这一步可以通过修改php.ini文件实现：
	
	#默认使用文件存储
	;php_value[session.save_handler] = files
	#修改为使用memcache存储，内部使用memcache类存取数据
	php_value[session.save_handler] = memcache
	
	#文件存储的目录
	;php_value[session.save_path] = /var/lib/php/session
	#修改为memcache服务器的地址
	php_value[session.save_path] = "tcp://localhost:11211"
	
至此所有配置修改好了。

第二步，给用户存session。

	#index.php
	<?php
		session_start();
		$_SESSION['name'] = 'zifeiwu.com';
		$_SESSION['birth'] = 1988;
	?>
	
最后一步，验证。

然后使用浏览器访问这个页面，并使用firebug查看是否有sessionid这个cookie：

 ![sessoonid](/images/image/2013/9-12-1.png)

可以看到sessionid为： 2jeo1rcikp6823gucu9b4lr3d3  

然后，我们连接到memcache服务器，获取这个值：
	
	#登录到memcache服务器
	telnet 127.0.0.1 11211
	
	#显示如下，说明登录进来了
	Trying 127.0.0.1...
	Connected to 127.0.0.1.
	Escape character is '^]'.
	
	#获取session的值
	get 2jeo1rcikp6823gucu9b4lr3d3
	
	#下面是结果：
	name|s:11:"zifeiwu.com";birth|i:1988;
	END
	
我们从memcache服务器上拿到的值确实是我们存在session中的值，说明php现在确认已经开始使用memcache存取session了。
	

<!--上面我们已经成功的使php用memcache存储session了，但是，缺点也很明显：我们是通过修改php.ini文件实现的，而不是通过程序去控制。-->



	





