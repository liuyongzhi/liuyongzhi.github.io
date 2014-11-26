---
layout: post
category: "Jenkins"
title:  "Jenkins的安装步骤"
tags: [Jenkins]
---
&nbsp;&nbsp;&nbsp;&nbsp;  Jenkins是一款Java平台的开源持续集成（Continuous Integration，CI）引擎。它易于安装，配置简单，丰富的插件支持，高度的可扩展性，强大的分布式构建能力都让它在众多的CI引擎中脱颖而出。主页：[http://jenkins-ci.org/](http://jenkins-ci.org/)

##一、安装方法主要有两种
###1.1、war包方式

对于linux和windows，直接下载war并放置到server(eg tomcat)下面，直接运行即可

###1.2、通过java -jar jenkins.war方式运行

##二、Jenkins在Linux下的安装步骤
linux平台下面，使用yum进行安装，具体命令如下： 

**sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo <br/>
sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key  <br/>
yum install jenkins**

##三、Jenkins在Windows下的安装步骤
1. 下载jenkins.war, 拷贝到c:\jenkins下
- 然后运行java -jar jenkins.war. 
- 访问http://localhost:8080 