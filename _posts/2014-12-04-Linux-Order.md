---
layout: post
category: "Linux"
title:  "Linux常用指令"
tags: [Linux]
---
	
	ntsysv：启动服务配置界面，随系统自动启动
	chkconfig --list ：查看linux下所有服务
##一、Linux环境下用户、组的操作
**1、建用户：**、

	adduser phpq                             //新建phpq用户
	passwd phpq                               //给phpq用户设置密码

**2、建工作组**
	
	groupadd test                          //新建test工作组

**3、新建用户同时增加工作组**
	
	useradd -g test phpq                      //新建phpq用户并增加到test工作组
	注：：-g 所属组 -d 家目录 -s 所用的SHELL


**4、给已有的用户增加工作组**
	
	usermod -G groupname username
	或者：gpasswd -a user group

**5、临时关闭**：在/etc/shadow文件中属于该用户的行的第二个字段（密码）前面加上*就可以了。想恢复该用户，去掉*即可。
	
	或者使用如下命令关闭用户账号：
	passwd peter –l
	重新释放：
	passwd peter –u

**6、永久性删除用户账号**

	userdel peter
	groupdel peter
	usermod –G peter peter   （强制删除该用户的主目录和主目录下的所有文件和子目录）

**7、从组中删除用户**

	编辑/etc/group 找到GROUP1那一行，删除 A
	或者用命令
	gpasswd -d A GROUP

**8、显示用户信息**

	id user
	cat /etc/passwd

**9、查看用户所在组**

	groups user_name