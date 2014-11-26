---
layout: post
category: "分布式消息队列"
title:  "Curator-Framework开源Zookeeper快速开发框架介绍"
tags: [分布式消息队列,Zookeeper,Curator-Framework,框架]
---
Zookeeper 客户端框架 Curator-Framework 来自Netflix公司，现在归Apache，目前版本2.0.1！

在使用ZK开发时会遇到让人头疼的几个问题，ZK连接管理、SESSION失效等一些异常问题的处理，Curator替我们解决了这些问题，通过对ZK连接状态的监控来做出相应的重连等操作，并触发事件！

更好的地方是Curator对ZK的一些应用场景提供了非常好的实现，而且有很多扩充，这些都符合ZK使用规范。
主要组件:

1. Recipes， ZooKeeper的系列recipe实现, 基于 Curator Framework.
2. Framework， 封装了大量ZooKeeper常用API操作，降低了使用难度, 基于Zookeeper增加了一些新特性，对ZooKeeper链接的管理，对链接丢失自动重新链接。
3. Utilities，一些ZooKeeper操作的工具类包括ZK的集群测试工具路径生成等非常有用，在Curator-Client包下org.apache.curator.utils。
4. Client，ZooKeeper的客户端API封装，替代官方 ZooKeeper class，解决了一些繁琐低级的处理，提供一些工具类。
5. Errors，异常处理, 连接异常等
6. Extensions，对curator-recipes的扩展实现，拆分为curator-:stuck_out_tongue_closed_eyes:iscovery
7. curator-:stuck_out_tongue_closed_eyes:iscovery-server提供基于RESTful的Recipes WEB服务.

其中Curator-Recipes包括有Elections（领导选举）、Locks（锁）、Queues（队列）、Barriers（屏障）、Counters（共享计数器）、Caches（状态管理，可用做配置管理、缓存等）
实践

官方提供学习实例curator-examples，可通过Maven下载，curator系列Maven地址 ：[maven:org.apache.curator](maven:org.apache.curator)

基于Curator做几个常用场景的开发实例：

**配置管理：** [http://www.cnblogs.com/xguo/archive/2013/06/10/3130589.html](http://www.cnblogs.com/xguo/archive/2013/06/10/3130589.html)

**分布式队列处理：** [http://www.cnblogs.com/xguo/archive/2013/06/10/3130589.html](http://www.cnblogs.com/xguo/archive/2013/06/10/3130589.html)

