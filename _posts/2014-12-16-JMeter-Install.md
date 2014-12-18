---
layout: post
category: "JMeter"
title:  "JMeter安装说明"
tags: [JMeter]
---  

##一、安装步骤：
	
###1.1、安装包准备：

	JDK1.5.0_12 具体可在官方网站下载
	jakarta-jmeter-2.3.4 具体下载地址：http://jakarta.apache.org/

###1.2、安装过程：

**1）、JDK安装 **
	
	点击下载的jdk-1_5_0_12-windows-i586-p.exe，选择安装路径即可。

**2）、JDK环境配置 **
	
	桌面上选择“我的电脑”(右键)/高级/环境变量, 在“系统变量”栏中点击“新建”, 
	在变量名中输入：CLASSPATH，变量值中输入：C:\JDK安装目录\lib\dt.JAR; 
	C:\JDK安装目录\lib\TOOLS.JAR;点击确定即可。
	
	再按“新建”，在变量名中输入：java_home，变量中输入：C:\JDK安装目录；
	修改PATH变量，添加% java_home %\bin；然后确定即可。
	
	修改系统变量path的值，在前面增加%java_home%\bin;然后确定即可。

**3）检查JDK安装是否OK**
	
	具体是：点击“开始”/“运行”，输入命令cmd进入dos操作界面，输入命令：java –version 查看java版本，如果显示为：1.5.0_12，则安装OK

**4）Jmeter安装**
	
	解压jakarta-jmeter-2.3.4文件至c盘，本文解压至C:\jmeter-2.3.4目录下。
	桌面上选择“我的电脑”(右键)/高级/环境变量, 在“系统变量”栏中点击“新建”, 
	在变量名中输入：JMETER_HOME，变量值中输入：C:\ jmeter-2.3.4，点击确定即可。
	
	再修改CLASSPATH变量，变量值中添加如下值：
	
	%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;%JMETER_HOME%\lib\logkit-1.2.jar; 然后确定即可。

**5）检查jmeter安装是否OK**
	
	具体是：进入jmeter目录下的bin文件夹，点击jmeter.bat，查看页面显示，如果能显示jmeter操作页面则安装成功。
	
	提醒：通常安装到这一步会报下面这个错误：
	
	unrecognized vm option '+heapdumponoutofmemoryerror'
	
	原因是：安装的JDK版本是：1.5.0的就会出错，把JDK卸载，重新下载JDK1.5.0_12版本，重新安装就OK了，
	之前我就是装的1.5.0版本报的错，后来更新JDK版本就好了。

 
