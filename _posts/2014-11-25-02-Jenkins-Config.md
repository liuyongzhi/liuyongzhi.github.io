---
layout: post
category: "Jenkins"
title:  "Jenkins配置"
tags: [Jenkins]
---
##一、系统管理
在已运行的Jenkins主页中，点击左侧的系统管理进入如下界面：

!["图1、Jenkins系统管理"](/images/Jenkins/2014-11-25-02-Jenkins-System-Manage.png "图1、Jenkins系统管理")

###1.1、提示信息
**注意**：版本不同提示的消息有可能不同

####1.1.1、Utf-8编码

Your container doesn't use UTF-8 to decode URLs. If you use non-ASCII characters as a job name etc, this will cause problems. See Containers and Tomcat i18n for more details.

Jenkins建议在tomcat中使用utf-8编码，配置tomcat下conf目录的server.xml文件

!["图1.1.1、Utf-8编码"](/images/Jenkins/2014-11-25-02-Jenkins-UTF-8-Code.png "图1.1.1、Utf-8编码")

**注意**：如果Job的控制台中文输出乱码，请将URIEncoding=”utf-8”更改为useBodyEncodingForURI="true"
####1.1.2、新的版本

New version of Jenkins (1.518.JENKINS-14362-jzlib) is available for download (changelog).

提示有新的版本可以下载了,喜欢更新的点击download去下载吧！
####1.1.3、安全设置
!["图1.1.2、安全提示消息"](/images/Jenkins/2014-11-25-02-Jenkins-Safe-Alert.png "图1.1.2、安全提示消息")

詹金斯允许网络上的任何人代表您启动进程。考虑至少启用身份验证来阻止滥用。点击Dismiss忽略该消息,点击Setup Security进入设置界面.详细设置请参考 Configure Global Security(安全设置) 章节

###1.2、系统设置

在已运行的Jenkins主页中，点击左侧的系统管理—>系统设置进入如下界面：

图6 系统设置界面
####1.2.1、JDK、Maven、Ant配置

配置一个JDK、Ant、Maven实例，请在每一节下面单击Add(新增) 按钮，这里将添加实例的名称和绝对地址。下图描述了这两个部分。

 

图7 JDK配置界面

JDK别名：给你看的，随便你自己，叫阿猫阿狗都可以

JAVA_HOME：这个是本机JDK的安装路径（错误的路径会有红字提示你的）

自动安装：不推荐这个选项

后面Ant与Maven的配置是一样的，JDK去oracle官网下载，Ant与Maven去apache官网下载

Ps：每个文本框后面都有个问号，点击问号就会出现帮助信息
####1.2.2、邮件通知配置
**配置发件人地址**

 

 

图8 发件人地址配置界面

System Admin e-mail address：Jenkins邮件发送地址，如果你这个没有配置，等着发邮件的时候报错吧，当时我也是这儿没有配置，郁闷了我一周的时间。⊙﹏⊙b汗 

**配置邮件通知**

 

图9 邮件通知

这个就非常的简单了，根据的的邮箱提供者的参数配置就行了。

Ps：小技巧：用户默认邮件后缀配置了后，以后你填写邮件地址只需要@之前的就行了
####1.2.3、Subversion配置

 

图10 Subversion配置

Subversion Workspace Version：Subversion 的版本号，选择你对应的版本号就行了
###1.3、Configure Global Security(安全设置)

在已运行的Jenkins主页中，点击左侧的系统管理—>Configure Global Security进入如下界面：

 

图11 安全设置界面

设置如上图，保存后系统管理中就出现管理用户的选项。页面右上角也会出现登录/注册的选项。
###1.4 管理用户设置

在右上角点击注册

 

图12 注册用户界面

点击sign up按钮，提示你现在已经登录.返回首页.

登录后和匿名账号看到的首页有几点不同，如下图红框所示：

 

图13 用户登录界面

**Jenkins安装和入门系列：**
[http://blog.csdn.net/wangmuming/article/category/2167947](http://blog.csdn.net/wangmuming/article/category/2167947)