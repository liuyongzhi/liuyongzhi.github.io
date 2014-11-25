---
layout: post
category: "Memcached"
title:  "Java语言实现Memcached缓存技术"
tags: [Memcached]
---
##、在windows下实现Memcached缓存技术步骤

1. 下载**memcached-1.2.1-win32.zip**并且解压.
2. 在项目中导入必要的包：

	alisoft-xplatform-asf-cache-2.5.1.jar

	commons-logging-1.0.3.jar

	log4j-1.2.13.jar

3. 创建工具类MemCachedUtil
4. 新建一个配置文件MemCached.xml

##一、Windows安装memcached
###1.1、Memcached-win64 下载：


1. 下载最新版：[http://blog.couchbase.com/memcached-windows-64-bit-pre-release-available](http://blog.couchbase.com/memcached-windows-64-bit-pre-release-available)
2. 直接下载： memcached-win64-1.4.4-14.zip
[http://www.2cto.com/uploadfile/2012/0713/20120713110308123.zip](http://www.2cto.com/uploadfile/2012/0713/20120713110308123.zip)

###1.2、解压放某个盘下面，比如:

D:\WampServer\bin\memcached\memcached.exe

###1.3. 在终端（也即cmd命令界面）下输入以下命令安装windows服务:

D:\WampServer\bin\memcached>memcached.exe -d install

###1.4、再输入下面命令启动：

D:\WampServer\bin\memcached>memcached.exe -d start

##二、Java实现Memcached缓存
###2.1、向POM.xml中添加如下代码
	<!-- xmemcached缓存 -->
		<dependency>
			<groupId>com.googlecode.xmemcached</groupId>
			<artifactId>xmemcached</artifactId>
			<version>2.0.0</version>
		</dependency>

###2.2、添加一个管理类MemcachedManager

	public class MemcachedManager {
		private static MemcachedClient mcc;
		static {
			try {
				String ip = "192.168.6.195";//设置memcached所在的Ip地址
				int port = 20000;//设置memcached的端口号
				if (mcc == null) {
					mcc = new XMemcachedClient(ip, port);
				}
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		public static MemcachedClient getInstance() {
			return mcc;
		}
	}

###2.3、查询数据

!["memcached_achieve03.png"](/images/Memcached/memcached_achieve03.png)

###2.4、更新数据

!["memcached_achieve04.png"](/images/Memcached/memcached_achieve04.png)

##三、Memcached现状
	1、支持更多协议，在已有协议支持的基础上添加了append、prepend、gets、批量gets、cas协议的支持，具体请查看XMemcachedClient类的实例方法。重点是cas操作。
	2、memcached分布支持，支持连接多个memcached server，支持简单的余数分布和一致性哈希分布。
	3、0.60版本以来的bug修复。
	当前1.0-beta版本，支持memcached的分布式（余数哈希和一致性哈希算法）。目前已经支持get、set、add、replace、cas、 append、prepend、批量get/gets、delete、incr、decr、version这几个协议。API为阻塞模型，而非 spymemcached的异步模型，异步模型在批处理的时候有优势，但是阻塞模型在编程难度和使用上会容易很多。
##四、不理解的知识点
	1、使用什么软件测试
	2、怎么使用java实现Memcached缓存技术
	3、MemCached怎么实现列表的缓存
	4、数据库中增加、删除、修改数据时，缓存如何及时的反应