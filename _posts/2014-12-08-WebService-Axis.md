---
layout: post
category: "Web Service"
title:  "Web Service 使用 Apache Axis2"
tags: [Web Service,Axis2]
---
Axis是apache下一个开源的webservice开发组件，出现的算是比较早了，也比较成熟。这里主要介绍Axis+eclipse开发webservice，当然不用eclipse也可以开发和发布webservice，只是用eclipse会比较方便。

##一、eclispe中Axis的配置

###（1）下载eclipse的Java EE版本
	
	http://www.eclipse.org/downloads/

###（2）下载axis2
	
	http://axis.apache.org/axis2/java/core/download.cgi

###（3）下载eclipse的axis2插件
	
	Axis2_Codegen_Wizard
	Axis2_Service_Archiver
下载地址：[http://axis.apache.org/axis2/java/core/tools/index.html](http://axis.apache.org/axis2/java/core/tools/index.html)

###（4）eclipse安装axis2插件

    1）在任意目录下新建一个Axis2文件夹，在该文件夹下新建eclipse目录，
	   在eclipse目录中新建plugins目录和features目录，例如：D:\programSoftware\eclipse-SVN\A\eclipse；
    2）把下载的axis2插件解压，并把解压的文件放到新建的eclipse的plugins目录下；
    3）在Eclipse_home%的目录下新建links目录，并在links目录下新建axis2.link文件，内容为：   
		 path=D:\programSoftware\eclipse-SVN\Axis2;
    4）重启eclipse，点击·file-new-other,如果看到Axis2 Wizards，则表明插件安装成功。

###（5）安装axis2
	
	下载Axis2的WAR Distribution并解压，把axis2.war包放置到%TOMCAT_HOME%/webapps下，
	启动tomcat，访问http://localhost:port/axis2，Axis2安装成功。

###（6）使用eclipse新建web工程，创建一个普通java类，至少包含一个方法。

###（7）发布webservice

    1）点击eclipse的File-New-other，打开Axis2 Wizards，选择Axis2 Service Archiver，然后Next;
    2）选择Class File Location，也就是类文件存放路径，注意：只选到classes目录，不要包括包文件夹，然后Next;
    3）选择Skip WSDL，然后Next
    4）一路Next到Select the Service XML file to be included in the Service archive，
		勾选Generate the service xml automatically；
    5）Service Name-填写你的service名称，Class Name-填写类名称，要包括包名，然后点击load，
		然后点击Finish，这时webservice就发布成功了；
    6）然后到%TOMCAT_HOME%/webapps/axis2/WEB-INF/services 看看是否多了一个.aar的文件；
    7）访问http://localhost:8085/axis2/services/类名?wsdl 就可看到生成的wsdl文件了。

 
**注意：**以上的方式是发布到axis2.war包中，你也可以把生成.aar文件copy到你的实际应用中，同时，你也可以使用eclipse的create webservice功能发布你的webservice，选择axis2生成你的webservice，这样webservice就会部署到你的应用中了