---
layout: post
category: "Linux"
title:  "Linux环境下Samba服务器的配置"
tags: [Linux,Samba]
---
##一、Samba服务器的配置步骤
	1、安装有关Samba的RPM包(samba、samba-common、samba-client)
	2、创建Samba用户
	3、修改配置文件
	4、重启samba服务
	5、设置目录访问权限
	6、测试
###1.1、安装RPM包(缺省情况下安装了samba的相关软件包，可以用如下命令查看)
	[root@localhost ~]#rpm -qa | grep samba
	强制安装：-f       
	跳过依赖安装：--nodeps
	samba -----samba服务器程序的所有文件
	samba-common -----提供了Samba服务器和客户机中都必须使用的公共文件
	samba-client -----提供了Samba客户机的所有文件
	samba-swat -----以Web界面的形式提供了对Samba服务器的管理功能
###1.2、创建samba用户
	[root@localhost ~]#smbpasswd -a user1 
	（“-a”是创建samba用户，“-x”是删除samba用户，“-d”是禁用samba用户帐号，“-e”是启用samba用户帐号）
	[root@localhost ~]#smbpasswd -a user2 
	[root@localhost ~]#smbpasswd -a user3 

###1.3、修改配置文件

	samba配置文件的位置：/etc/samba/smb.conf
	[root@localhost ~]#vim /etc/samba/smb.conf

**Samba服务器的安全级别**

	Vi大开配置文件后，首先介绍一下Samba服务器的安全级别，如图所示：系统默认设置“user”
	Samba服务器的安全级别分为5种，分别是user、share、server、domain和ads。在设置不同的级别时，samba服务器还会使用口令服务器和加密口令。
	1、user -----客户端访问服务器时需要输入用户名和密码，通过验证后，才能使用服务器的共享资源。此级别使用加密的方式传送密码。
	2、share -----客户端连接服务器时不需要输入用户名和密码
	3、server -----客户端在访问时同样需要输入用户名和密码，但是，密码验证需要密码验证服务器来负责。
	4、domain -----采用域控制器对用户进行身份验证
	5、ads -----若samba服务器加入到Windows活动目录中，则使用ads安全级别，ads安全级别也必须指定口令服务器

**共享目录的配置**

	[homes] -----samba用户的宿主目录
	comment = Home Directories -----设置共享的说明信息
	browseable = no -----目录浏览权限
	writable = yes -----用户对共享目录可写
	这个共享目录只有用户本身可以使用，默认情况下，用户主目录位于/home目录下，每个Linux用户有一个以用户名命名的子目录。
	以下是共享打印机的设置：

	[printers] -----共享打印机
	comment = All Printers -----设置共享的说明信息
	path = /var/spool/samba -----指定共享目录的路径
	browseable = no -----目录浏览权限
	guest ok = no -----允许来宾访问
	writable = no -----用户对共享目录可写
	printable = yes -----可以打印
	以上是系统默认设置

	添加自定义的共享目录：( user1对/ASUS有所有权，user2拥有只读权限，其他用户不能访问；public共享目录允许所有用户访问及上传文件)
	[ASUS]
	comment = user1 Directories -----设置共享的说明信息
	browseable = yes -----所有samba用户都可以看到该目录
	writable = yes -----用户对共享目录可写
	path = /ASUS -----指定共享目录的路径

	[public]
	comment = all user Directories -----设置共享的说明信息
	browseable = yes -----所有samba用户都可以看到该目录
	writable = yes -----用户对共享目录可写
	path = /public -----指定共享目录的路径
	guest ok = yes -----允许来宾访问

###1.4、修改完配置文件后需要重启samba服务
	[root@localhost ~]#service smb restart

	Samba服务器包括两个服务程序:
	(1)、smbd
	smbd服务程序为客户机提供了服务器中共享资源的访问
	(2)、nmbd
	nmbd服务程序提供了NetBIOS主机名称的解析，为Windows网络中域或者工作组内的主机进行主机名称的解析
###1.5、设置目录权限
	[root@localhost ~]#mkdir /ASUS ------创建要共享目录
	[root@localhost ~]#mkdir /public ------创建要共享的目录
	[root@localhost ~]#chmod 750 /ASUS ------修改/ASUS权限（属主拥有所有权，属组只读，其它用户不能访问）
	[root@localhost ~]#chown user1 /ASUS ------将/ASUS的属主改为user1
	[root@localhost ~]#groupadd ASUS ------添加ASUS组
	[root@localhost ~]#usermod –G ASUS user1 ------将user1加入到ASUS组
	[root@localhost ~]#usermod –G ASUS user2 ------将user2加入到ASUS组
	[root@localhost ~]#chgrp ASUS /ASUS ------将/ASUS的属组改为ASUS
	[root@localhost ~]#chmod 777 /public ------给所有用户分配完全控制
	权限

配置完成后，还要检查/etc/service文件中以“netbios”开头的记录，正确的文件记录如下所示，如果这些记录前有#或没有这些记录，应手工添加，否则用户无法访问Linux服务器上的共享资源

	netbios-ns      137/tcp                         # NETBIOS Name Service
	netbios-ns      137/udp
	netbios-dgm     138/tcp                         # NETBIOS Datagram Service
	netbios-dgm     138/udp
	netbios-ssn     139/tcp                         # NETBIOS session service
	

###1.6、测试
找一台内网windows客户端，打开“网上邻居 ”，输入samba服务器的IP点击“搜索”

##二、linux samba服务启动正常，window无法访问
	
	1.首先确定linux和windows的防火墙都关了
	2.然后确定linux和windows之间的网络是不是通的，可用ping命令检测
	3.最后用“smbpasswd -a 用户名”命令添加samba用户，就可以在windows中访问了

##三、可以登录samba服务器，但是没有权限访问linux下的共享目录
	
	1、确保linux下防火墙关闭或者是开放共享目录权限  iptalbes -F
	2、确保samba服务器配置文件smb.conf设置没有问题，可网上查阅资料看配置办法
	3、确保setlinux关闭，可以用setenforce 0命令执行。 默认的，SELinux禁止网络上对Samba服务器上的共享目录进行写操作，即使你在smb.conf中允许了这项操作。
	这两个命令必须执行啊：
	   iptables -F
	   setenforce 0

##参考网址：
Samba服务器的配置：[http://yangxuejun.blog.51cto.com/623927/180224](http://yangxuejun.blog.51cto.com/623927/180224)