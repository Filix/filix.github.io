---
layout: post
title: MySql基本的安全设置
tag: [database]
keywords: 子非吾,MySql,security setting,技术博客,it
---
#MySql基本的安全设置
转自：<http://www.lvtao.net/database/mysql-safe.html>

###1.设置或修改Mysql root密码：
默认安装后空密码，以mysqladmin命令设置密码：

	mysqladmin -uroot password "password"

Mysql命令设置密码：

	mysql> set password for root@localhost=password('password);

更改密码：
	
	update mysql.user set password=password('password') where user='root';
	flush privileges;

###2.删除默认的数据库和用户
	drop database test;
	use mysql;
	delete from db;
	delete from user where not(host="localhost" and user="root");
	flush privileges;
###3. 更改默认root账号名称：

	update mysql.user set user="admin" where user="root";
	flush privileges;

###4. 本地文件安全：

	set-variable=local-infile=0
###5. 禁止远程连接mysql
编辑my.cnf在[mysqld]添加：

	skip-networking

###6.最小权限用户：

	create database db1;
	grant select,insert,update,delete,create,drop privileges on database.* to
	user@localhost identified by 'passwd';

###7. 限制普通用户浏览其它数据库
编辑my.cnf在[mysqld]添加：
	
	--skip-show-database

###8.快速修复MySQL数据库
修复数据库

	mysqlcheck -A -o -r -p
	
修复指定的数据库

	mysqlcheck  -o -r database -p
###9.跟据内存的大小选择MySQL的配置文件

	my-small.cnf # > my-medium.cnf # 32M - 64M
	my-large.cnf # memory = 512M
	my-huge.cnf # 1G-2G
	my-innodb-heavy-4G.cnf # 4GB
	