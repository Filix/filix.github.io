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

###使用memcache类管理session

第一步，修改php的session处理器，使之从默认的文件变为memcache。    
这一步可以通过修改php.ini文件实现：
	
	#默认使用文件存储
	;session.save_handler = files
	#修改为使用memcache存储，内部使用memcache类存取数据
	session.save_handler = memcache
	
	#文件存储的目录
	;session.save_path = /var/lib/php/session
	#修改为memcache服务器的地址
	session.save_path = "tcp://localhost:11211"
	
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

当然我们还可以通过ini_set函数修改php的配置来实现此功能：

	ini_set("session.save_handler", "memcache");
	ini_set("session.save_path", "tcp://127.0.0.1:11211");  
其思路都是一样的。
	
###使用session_set_save_handler实现
上面我们已经成功的使php用memcache存储session了，但是，缺点也很明显：我们是通过修改php.ini文件或者修改php默认配置实现的，而不是通过程序去控制。如果有的时候默认的行为不能满足我们的需求时，就不得不得去修改php的行为了。    

php为我们提供了在程序中控制session存取的方法：session_set_save_handler()函数。通过使用这个方法，我们可以100%的掌控session的每一个环节，比如添加额外的信息、修改默认的key名等等。    
	
	bool session_set_save_handler ( callable $open , callable $close , callable $read , callable $write , callable $destroy , callable $gc )
	
	#此方式要求php>=5.4.0，SessionHandlerInterface接口在5.4才加入
	bool session_set_save_handler ( SessionHandlerInterface $sessionhandler [, bool $register_shutdown = true ] )
	
从上面可以看出来，这个函数有两种使用方法：

1. 提供6个管理session的callable方法，这6个方法包括：实例化/打开/连接session的处理器，关闭session处理器，从处理器中读session，写session数据到处理器，销毁/删除session数据，清理session的方法。    
 说的明白一点，还是以使用memcache为例。这6个方法分别定义了：如何连接memcache、如何关闭到memcache的连接、如何从memcache中读取一个sessionid对应的数据、如何写数据到一个sessionid中、如何从memcache中删除一个session、如何清除过期的session。
 
2. 实现SessionHandlerInterface接口。其实SessionHandlerInterface也就定义了上面说的6个方法。


下面我们就实现一次上面的第二种方法。

	
	# MemcacheSessionHandler.php
	<?php
 	
    class MemcacheSessionHandler implements SessionHandlerInterface{
        
        private $memcache;
        #session过期时间，可以通过程序去控制，为0的话，永远不过期，因为memcache->set()的时候过期时间为0的话永不过期
        private $gc_maxlifetime = 999999;
        
        public function __construct(){
            $this->memcache = new Memcache();
            $this->memcache->connect('localhost', 11211);
        }
        
        public function open ($save_path, $name){
            return true;
        }

		#读取session的值
        public function read($session_id){
            return $this->memcache->get($this->getKey($session_id));
        }

		#保存session值
        public function write($session_id, $session_data){
            $a = $this->memcache->set($this->getKey($session_id), $session_data, null, $this->getGcMaxLifetime());
            return $a;     
        }
	
		#过期session的回收，因为memcache中设置了过期时间之后，会自动过期，所以这里不需要任何处理，不像文件或者数据库，还需要遍历所有session，删除过期的session
        public function gc($maxlifetime){
			return true;
        }
        
        #删除某个session的值
        public function destroy($session_id){
            $this->memcache->delete($this->getKey($session_id));
        }
		
		#可以自定义key
        protected function getKey($session_id){
            return "mem.sess." . $session_id;
        }
               
        #过期时间                                           
        protected function getGcMaxLifetime(){            
            return $this->gc_maxlifetime !== null ? $this->gc_maxlifetime : ini_get('session_gc_maxlifetime');
        }
        
        public function close(){
            unset($this->memcache);
        }
    }
    
 然后，调用session_set_save_handler()
 
 
 	# index.php
 	<?php
 		include_once('./MemcacheSessionHandler.php');
    	$session_handler = new MemcacheSessionHandler();
    	session_set_save_handler($session_handler, true);
    	session_start();
    	if(empty($_SESSION)){
        	$_SESSION['name'] = 'zifeiwu.com';
        	$_SESSION['birth'] = 1988;
    	}else{
        	var_dump($_SESSION);
    	}
    ?>
    
验证过期和上面是一样一样的，自己试试吧。
		


###使用memcached类管理session
前面介绍了，使用memcache类管理session。前面也说了，php有两个memcache扩展，现在介绍下memcached类的使用方法。

如果是通过修改php.ini实现的，只是在配置方面有点下的区别：

	#使用memcache
	;session.save_handler = memcache
	#修改为memcached，则使用memcached类实现
	;session.save_handler = memcached
	
	#memcache服务器的地址
	session.save_path = "tcp://localhost:11211"
	#没有tcp://
	session.save_path = "localhost:11211"
	

###使用多台memcache服务器
当我们的应用足够大的时候，单台memcache实例已经不能满足我们的需求了，不足以保存所有的session时，自然而然就要考虑增加memcache服务器。这时候在php.ini文件中使用逗号分割服务器：

	session.save_path = "192.169.0.1:11211;192.168.0.2:11211"

对于多台缓存服务器的情况，虽然手册上没提到session的哈希一致性的问题（就是一个用户的session会固定到一台服务器上），但是肯定在内部已经实现了。    
如果采用session_set_save_handler的方式，肯定要自己去实现哈希一致性了。   
另外，对于使用memcache存取session时，建议单台实例只做这件事，不要存取其余的数据在该台实例上，以免内存不足时，memcache自动丢掉用户的session而造成用户莫名掉线的问题。


写了这么多，你是不是也该试试了？试试Mysql或者Redis如何？




