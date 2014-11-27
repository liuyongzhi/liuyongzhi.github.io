---
layout: post
category: "Zookeeper"
title:  "ZooKeeper安装"
tags: [Zookeeper,安装]
---
ZookKeeper 是Apache的顶级项目之一，我们可用它来做配置管理、集群管理 和 消息队列等。

安装相对简单，同样有单机和分布式俩种安装模式，单机少一些配置，本文给出安装方法和一些常用管理命令等。
##一、环境要求

    linux 或者 windows（生产环境最好不用）

    jdk1.6 以上

    设置Java heap 大小。避免内存与磁盘空间的交换，能够大大提升ZK的性能，
		设置合理的heap大小则能有效避免此类空间交换的触发。在正式发布上线之前，
		建议是针对使用场景进行一些压力测试，确保正常运行后内存的使用不会触发此类交换。
		通常在一个物理内存为4G的机器上，最多设置-Xmx为3G。

    下载ZK，http://zookeeper.apache.org/releases.html
    并解压到usr/local/zookeeper
    三台虚拟机：
    server.1=zoo1:2888:3888
    server.2=zoo2:2888:3888
    server.3=zoo3:2888:3888

##二、Zookeeper配置

编辑配置文件，在conf目录：
	
	move ./zoo_sample.cfg ./zoo.cfg
	vim zoo.cfg

内容如下：
	
	tickTime=2000
	dataDir=/var/lib/zookeeper/data
	clientPort=2181
	initLimit=5
	syncLimit=2
	
	##单机不需要以下配置
	server.1=zoo1:2888:3888
	server.2=zoo2:2888:3888
	server.3=zoo3:2888:3888

设置Myid，单机不需要

**server.id=host:port:port ** 的格式，server固定，后面的数字是此服务器在集群中的序号[1~255],并需要data目录下创建文件myid来存储当前server的id。

设置服务器id：
	
	echo 1 >> /var/lib/zookeeper/data/myid

其他俩台分别执行：
	
	echo 2 >> /var/lib/zookeeper/data/myid
	和
	echo 3 >> /var/lib/zookeeper/data/myid

##三、启动ZK
	
	cd /usr/local/zookeeper/bin
	./zkServer.sh start (windows下是zkServer.cmd)

查看启动状态：
	
	./zkServer.sh status

通过ZK客户端连接ZK
	
	./zkCli.sh 或者 ./zkCli.sh -server 127.0.0.1:2181

需要注意的是在集群模式下，ZK不能像Hadoop那样只需要在master下执行启动命令Hadoop自动启动slaves，ZK需要手动去启动每个节点！

至此，ZK安装配置结束！
##四、Zookeeper配置参数说明

    tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，单位毫秒。

    dataDir：是 Zookeeper 保存数据和日志的目录。

    clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口。

    initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，
	而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。
	当已经超过 5个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，
	那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒

    syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，
	最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒

    server.id=host：port1：port2：其中 id 是一个数字，表示这个是第几号服务器；
    host 是这个服务器的 ip 地址；
    port1 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；
    port2 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

##五、JMX远程支持

Zookeeper默认开启JMX，只限本地连接，远程连接需要修改启动脚本

打开 ./bin/zkServer.sh 找到：
	
	ZOOMAIN="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=$JMXLOCALONLY org.apache.zookeeper.server.quorum.QuorumPeerMain"

加入启动参数：
	
	-Dcom. sun.management.jmxremote.port=5000 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false

如：
	
	ZOOMAIN="-Dcom. sun.management.jmxremote.port=5000 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false org.apache.zookeeper.server.quorum.QuorumPeerMain"
##六、Zookeeper四字命令

ZK服务提供了下列命令让管事员来测试服务的运行情况。

    stat: 查看ZK结点follower或leader情况。
    srst: 重置通过stat查看的统计结果信息。
    ruok: 测试Server，若回复imok表示已经启动。
    dump: 列出未经处理的会话和临时节点。
    kill: 关掉server, 必须在ZK服务运行的机器上执行。
    conf: 输出相关服务配置的详细信息。
    cons: 列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。
    crst: 重置 connection/session 所有连接的统计信息。
    envi: 输出关于服务环境的详细信息。
    reqs: 列出未经处理的请求。
    wchs: 列出服务器 watch 的详细信息。
    wchc: 通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表。
    wchp: 通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径。
    srvr: 列出服务的详细信息。
    mntr: 打印所有可以用来监控集群服务健康状态信息的变量。

**用法：**
	linux： echo mntr|nc 127.0.0.1 2181
	
	zk_version  3.4.0
	zk_avg_latency  0
	zk_max_latency  0
	zk_min_latency  0
	zk_packets_received 70
	zk_packets_sent 69
	zk_outstanding_requests 0
	zk_server_state leader
	zk_znode_count   4
	zk_watch_count  0
	zk_ephemerals_count 0
	zk_approximate_data_size    27
	zk_followers    4                   - only exposed by the Leader
	zk_synced_followers 4               - only exposed by the Leader
	zk_pending_syncs    0               - only exposed by the Leader
	zk_open_file_descriptor_count 23    - only available on Unix platforms
	zk_max_file_descriptor_count 1024   - only available on Unix platforms

windows: telnet 127.0.0.1 2181 然后输入：stat等四字命令。

更多ZK管理内容请参考：[http://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html](http://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html)
##七、Zookeeper客户端常用命令

进入zkCli后敲入help，或者随便敲几个字符ZK会打印出如下信息：
	
	get path [watch]
	ls path [watch]
	set path data [version]
	rmr path
	delquota [-n|-b] path
	quit 
	printwatches on|off
	create [-s] [-e] path data acl
	stat path [watch]
	close 
	ls2 path [watch]
	history 
	listquota path
	setAcl path acl
	getAcl path
	sync path
	redo cmdno
	addauth scheme auth
	delete path [version]
	setquota -n|-b val path

**注：path 要以 / 打头；使用监听如：get /t watch**

**参考网址**

http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html
http://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html
http://www.cnblogs.com/haippy/archive/2012/07/19/2599989.html
