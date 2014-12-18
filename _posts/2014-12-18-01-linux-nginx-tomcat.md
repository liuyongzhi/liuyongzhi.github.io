---
layout: post
category: "集群"
title:  "linux环境下通过nginx实现tomcat集群"
tags: [linux,nginx,tomcat]
--- 

安装nginx之前需要pcre依赖和jvm-remote补丁
##一、准备如下软件：
	1、nginx-1.1.2.tar.gz，负载均衡/反向代理服务器，可通过http://nginx.org/en/download.html获取。
	2、pcre-8.10.tar.gz，正规表达式库，可通过http://sourceforge.net/projects/pcre/获取；
	3、nginx-upstream-jvm-route-0.1.tar.gz，是一个 Nginx 的扩展模块，用来实现基于 Cookie 的 Session Sticky 的功能，可通过http://code.google.com/p/nginx-upstream-jvm-route/downloads/list获取；

##二、安装和安装
###2.1、解压各软件
		
	[root@localhost ~]# tar zxvf pcre-8.10.tar.gz
	[root@localhost ~]# tar zxvf nginx-upstream-jvm-route-0.1.tar.gz
	[root@localhost ~]# tar nginx-1.1.2.tar.gz

###2.2、安装
	
	[root@localhost ~]# cd nginx-1.1.2
	[root@localhost ~]# patch -p0 < ${nginx-upstream-jvm-route解压目录}/jvm_route.patch
	[root@localhost ~]# ./configure --prefix=/usr/local/nginx --with-pcre=${pcre解压目录} --with-http_stub_status_module --with-http_ssl_module --add-module=${nginx-upstream-jvm-route解压目录}
	[root@localhost ~]# make
	[root@localhost ~]# make install

###三、修改配置
###3.1、修改tomcat的server.xml，服务器的tomcat的配置文件中分别找到：
<Engine name="Catalina" defaultHost="localhost" >
分别修改为：

	Tomcat01:
	<Engine name="Catalina" defaultHost="localhost" jvmRoute="a">
	Tomcat02:
	<Engine name="Catalina" defaultHost="localhost" jvmRoute="b">
	Tomcat03:
	<Engine name="Catalina" defaultHost="localhost" jvmRoute="c">

###3.2、修改nginx的nginx.conf文件
	
	#运行NGINX所使用的用户和组
	user  root;
	#nginx进程数，建议按照cpu数目来指定，一般为它的倍数，每个进程消耗约10M内存
	worker_processes  1;
	 
	#日志信息
	error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;
	 
	pid        logs/nginx.pid;
	 
	events {
	    #使用epoll的I/O模型
	    use epoll;
	    #该值受系统进程最大打开文件数限制，需要使用命令ulimit -n 查看当前设置
	    worker_connections  1024;
	}
	 
	 
	http {
	    #这里是您需要修改的地方，修改为您的服务器IP:端口号 srun_id为您在tomcat中所配置的jvmRoute
	    upstream backend{
	      server 192.168.12.128:18080 srun_id=a;
	      server 192.168.12.128:28080 srun_id=b;
	      server 192.168.12.128:38080 srun_id=c;
	      jvm_route $cookie_JSESSIONID|sessionid reverse;
	    }
	    include       mime.types;
	    #设置默认类型是二进制流，若未设置时，比如未加载PHP时，是不予解析，用浏览器访问则出现下载窗口
	    default_type application/octet-stream;
	    charset UTF-8;
	    server_names_hash_bucket_size 128;
	    client_header_buffer_size 32k;
	    large_client_header_buffers 4 32k;
	    client_max_body_size 20m;
	    limit_rate 1024k;
	    sendfile on;
	    tcp_nopush     on;
	    keepalive_timeout 60;
	    tcp_nodelay on;
	    fastcgi_connect_timeout 300;
	    fastcgi_send_timeout 300;
	    fastcgi_read_timeout 300;
	    fastcgi_buffer_size 64k;
	    fastcgi_buffers 4 64k;
	    fastcgi_busy_buffers_size 128k;
	    fastcgi_temp_file_write_size 128k;
	    gzip on;
	    #gzip_min_length 1k;
	    gzip_buffers     4 16k;
	    gzip_http_version 1.0;
	    gzip_comp_level 2;
	    gzip_types       text/plain application/x-javascript text/css application/xml;
	    gzip_vary on;
	    #limit_zone crawler $binary_remote_addr 10m;
	    server {
	        listen       80;
	        server_name  192.168.12.128; #这里也是您所需要修改的地方，多域名用空格隔开
	        index index.html index.htm index.jsp;
	        charset UTF-8;
	        root /usr/local/tomcats/project/;# 这里也是您所需要修改的地方，虚拟机指向的路径（可能这里有点问题），我的web应用系统放在project下面的
	        #access_log  logs/host.access.log  main;
	         
	        #这里也是您所需要修改的地方，yourproject更换成您的项目路径
	        location /yourproject/ {
	            proxy_pass http://backend;
	            proxy_redirect off;
	            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	            proxy_set_header X-Real-IP $remote_addr;
	            proxy_set_header Host $http_host;
	            index  index.html index.htm index.jsp;
	        }
	        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
	            expires 30d;
	        }
	        location ~ .*\.(js|css)?$ {
	            expires 1h;
	        }
	        location /Nginxstatus{
	            stub_status on;
	            access_log off;
	        }
	        log_format access '$remote_addr - $remote_user [$time_local] "$request" '
	             '$status $body_bytes_sent "$http_referer" '
	             '"$http_user_agent" $http_x_forwarded_for';
	 
	        error_page  404              /404.html;
	 
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }
	    }
	}


###3.3、检查nginx的配置
	
	[root@localhost ~]# /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf

##四、测试
###4.1、启动测试
		
	/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
	/usr/local/tomcats/tomcat-a/bin/startup.sh
	/usr/local/tomcats/tomcat-b/bin/startup.sh
	/usr/local/tomcats/tomcat-c/bin/startup.sh

###4.2、停止服务
		
	/usr/local/tomcats/tomcat-a/bin/shutdown.sh
	/usr/local/tomcats/tomcat-b/bin/shutdown.sh
	/usr/local/tomcats/tomcat-c/bin/shutdown.sh
	pkill -9 nginx
			