---
layout: post
category: "分布式消息队列"
title:  "Zookeeper和Curator-Framework实践系列之： 配置管理"
tags: [分布式消息队列,Zookeeper,Curator-Framework,配置管理]
---

看过Zookeeper相关文档后都知道它可以实现分布式集群的配置管理，本文以一个简单的实例来演示它是如何实现的并工作的。

##一、Zookeeper情景需要
情景需要，简单理解为下图：

!["messagequeue01"](/images/MessageQueue/messagequeue01.png)

一个web集群，需要通过zk来控制集群的日志输出级别，比如管理员需要在生产环境下查看一下DEBUG日志，他可以临时将集群的日志输出级别改为DEBUG，获取他想要的信息后还要将级别调回到INFO或者ERROR级别。

今天的主角是Curator-Framework，它的存在使得我们操作ZK变得简单有趣起来！
环境，框架，工具

    Zookeeper集群
    Zookeeper API
    Curator-Framework
    Spring-MVC
    Logback
    Maven + IDEA
##二、相关配置文件
###2.1、pom.xml：依赖关系和jetty插件
	
	<dependencies>
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-context</artifactId>
	        <version>3.2.3.RELEASE</version>
	    </dependency>
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-web</artifactId>
	        <version>3.2.3.RELEASE</version>
	    </dependency>
	    <dependency>
	        <groupId>org.springframework</groupId>
	        <artifactId>spring-webmvc</artifactId>
	        <version>3.2.3.RELEASE</version>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.zookeeper</groupId>
	        <artifactId>zookeeper</artifactId>
	        <version>3.4.5</version>
	        <exclusions>
	            <exclusion>
	                <groupId>log4j</groupId>
	                <artifactId>log4j</artifactId>
	            </exclusion>
	            <exclusion>
	                <groupId>org.slf4j</groupId>
	                <artifactId>slf4j-log4j12</artifactId>
	            </exclusion>
	        </exclusions>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.curator</groupId>
	        <artifactId>curator-framework</artifactId>
	        <version>2.0.1-incubating</version>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.curator</groupId>
	        <artifactId>curator-recipes</artifactId>
	        <version>2.0.1-incubating</version>
	    </dependency>
	    <dependency>
	        <groupId>ch.qos.logback</groupId>
	        <artifactId>logback-classic</artifactId>
	        <version>1.0.13</version>
	    </dependency>
	    <dependency>
	        <version>1.7.5</version>
	        <groupId>org.slf4j</groupId>
	        <artifactId>slf4j-api</artifactId>
	    </dependency>
	    <dependency>
	        <groupId>org.testng</groupId>
	        <artifactId>testng</artifactId>
	        <version>6.8.5</version>
	        <scope>test</scope>
	    </dependency>
	</dependencies>
	<build>
	    <finalName>zookeeper-configuration</finalName>
	    <plugins>
	        <plugin>
	            <groupId>org.mortbay.jetty</groupId>
	            <artifactId>jetty-maven-plugin</artifactId>
	            <version>8.1.7.v20120910</version>
	            <configuration>
	                <stopKey>pbase</stopKey>
	                <stopPort>8080</stopPort>
	            </configuration>
	        </plugin>
	    </plugins>
	</build>

###2.2、web.xml配置
	
	<web-app>
	  <display-name>Archetype Created Web Application</display-name>
	    <context-param>
	        <param-name>contextConfigLocation</param-name>
	        <param-value>
	            classpath*:/applicationContext.xml
	        </param-value>
	    </context-param>
	    <listener>
	        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	    </listener>
	    <servlet>
	        <servlet-name>springServlet</servlet-name>
	        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	        <init-param>
	        <param-name>contextConfigLocation</param-name>
	        <param-value>classpath*:/spring-mvc.xml</param-value>
	    </init-param>
	        <load-on-startup>1</load-on-startup>
	    </servlet>
	    <servlet-mapping>
	        <servlet-name>springServlet</servlet-name>
	        <url-pattern>/</url-pattern>
	    </servlet-mapping>
	</web-app>

###2.3、Spring 配置：applicationContext.xml
	
	<context:component-scan base-package="cn.bg">
	    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
	<!-- Curator的FactoryBean，Spring启动时创建Curator实例。 -->
	<bean id="zookeeperFactoryBean" class="cn.bg.zk.core.ZookeeperFactoryBean" lazy-init="false">
	    <property name="zkConnectionString" value="hadoopmaster:2181"/>
	    <!-- 设置zookeeper的事件监听者，本例是一个logback日志级别znode监听器 -->
	    <property name="listeners">
	        <list>
	            <bean class="cn.bg.zk.configuration.LogbackLevelListener">
	                <constructor-arg value="/zk_test/logbacklevel"/>
	            </bean>
	        </list>
	    </property>
	</bean>

###2.4、spring-mvc.xml配置
	
	<context:component-scan base-package="cn.bg">
	    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>
	<mvc:annotation-driven></mvc:annotation-driven>

##三、Zookeeper配置管理实现相关类

###3.1、ZookeeperFactoryBean.java

在Spring Context加载过程中创建Zookeeper链接对像并设置触发监听，通过Curator.
	
	package cn.bg.zk.core;

	public class ZookeeperFactoryBean implements FactoryBean<CuratorFramework>, InitializingBean, DisposableBean {
	
	    private Logger logger = LoggerFactory.getLogger(this.getClass());
	    private CuratorFramework zkClient;
	
	    //设置Zookeeper启动后需要调用的监听或者，或者需要做的初始化工作。
	    public void setListeners(List<IZKListener> listeners) {
	        this.listeners = listeners;
	    }
	
	    private List<IZKListener> listeners;
	
	    //设置ZK链接串
	    public void setZkConnectionString(String zkConnectionString) {
	        this.zkConnectionString = zkConnectionString;
	    }
	
	    private String zkConnectionString;
	
	    @Override
	    public CuratorFramework getObject() {
	        return zkClient;
	    }
	
	    @Override
	    public Class<?> getObjectType() {
	        return CuratorFramework.class;
	    }
	
	    @Override
	    public boolean isSingleton() {
	        return true;
	    }
	
	    @Override
	    public void destroy() throws Exception {
	        zkClient.close();
	    }
	
	    //创建ZK链接
	    @Override
	    public void afterPropertiesSet(){
	        //1000 是重试间隔时间基数，3 是重试次数
	        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
	        zkClient = createWithOptions(zkConnectionString, retryPolicy, 2000, 10000);
	        registerListeners(zkClient);
	        zkClient.start();
	    }
	
	
	    /**
	     * 通过自定义参数创建
	     */
	    public CuratorFramework  createWithOptions(String connectionString, RetryPolicy retryPolicy, int connectionTimeoutMs, int sessionTimeoutMs)
	    {
	        return CuratorFrameworkFactory.builder()
	                .connectString(connectionString)
	                .retryPolicy(retryPolicy)
	                .connectionTimeoutMs(connectionTimeoutMs)
	                .sessionTimeoutMs(sessionTimeoutMs)
	                .build();
	    }        
	
	    //注册需要监听的监听者对像. 
	    private void registerListeners(CuratorFramework client){
	        client.getConnectionStateListenable().addListener(new ConnectionStateListener() {
	            @Override
	            public void stateChanged(CuratorFramework client, ConnectionState newState) {
	                logger.info("CuratorFramework state changed: {}", newState);
	                if(newState == ConnectionState.CONNECTED || newState == ConnectionState.RECONNECTED){
	                    for(IZKListener listener : listeners){
	                        listener.executor(client);
	                        logger.info("Listener {} executed!", listener.getClass().getName());
	                    }
	                }
	            }
	        });
	
	        client.getUnhandledErrorListenable().addListener(new UnhandledErrorListener() {
	            @Override
	            public void unhandledError(String message, Throwable e) {
	                logger.info("CuratorFramework unhandledError: {}", message);
	            }
	        });
	    }
	}

###3.2、监听事件接口：IZKListener

所有需要在ZK客户端链接成功后需要做的事件，需要实现这个接口，由上面的ZookeeperFactoryBean统一调度。
	
	package cn.bg.zk.core;
	
	import org.apache.curator.framework.CuratorFramework;
	
	public interface IZKListener {
	    void executor(CuratorFramework client);
	}

###3.3、Logback监听实现

这里主要用到Curator的NodeCache类，它的主要功能是用来监听znode本身的变化，并可以获取当前值，而且会自动重复监听，简化了原生API开发的繁琐过程。
	
	package cn.bg.zk.configuration;
	
	public class LogbackLevelListener implements IZKListener {
	
	    //获取logback实例
	    Logger log = (Logger) LoggerFactory.getLogger(this.getClass());
	
	    private String path;
	
	    //Logback日志级别ZNode
	    public LogbackLevelListener(String path) {
	        this.path = path;
	    }
	
	    @Override
	    public void executor(CuratorFramework client) {
	
	        //使用Curator的NodeCache来做ZNode的监听，不用我们自己实现重复监听
	        final NodeCache cache = new NodeCache(client, path);
	        cache.getListenable().addListener(new NodeCacheListener() {
	            @Override
	            public void nodeChanged() throws Exception {
	
	                byte[] data = cache.getCurrentData().getData();
	
	                //设置日志级别
	                if (data != null) {
	                    String level = new String(data);
	                    Logger logger = (Logger) LoggerFactory.getLogger("root");
	                    Level newLevel = Level.fromLocationAwareLoggerInteger(Integer.parseInt(level));
	                    logger.setLevel(newLevel);
	                    System.out.println("Setting logback new level to :" + newLevel.levelStr);
	                }
	            }
	        });
	        try {
	            cache.start(true);
	        } catch (Exception e) {
	            log.error("Start NodeCache error for path: {}, error info: {}", path, e.getMessage());
	        }
	    }
	}

###3.4、通过WEB端查看当前日志级别

写一个controller来读取logback的日志级别：MainController.java

	package cn.bg.controller;
	
	@Controller
	public class MainController {
	
	    @RequestMapping("/")
	    @ResponseBody
	    public String logbackLevel() throws Exception {
	        Logger logger = (Logger) LoggerFactory.getLogger("root");
	        String levelStr = logger.getLevel().levelStr;
	        return levelStr;
	    }
	
	}

##四、运行

在ZK集群端启动zkCli创建/zk_test/logbacklevel的znode，设置值为10，或20在控制台来查看logback的日志输出情况，logback日志级别数据表示如下：
	
	TRACE_INT = 0
	DEBUG_INT = 10
	INFO_INT = 20
	WARN_INT = 30
	ERROR_INT = 40

到此就实现了一个简单的ZK配置管理情境，有了Curator-Framework后一切变的简单起来，我们主要精力只要解决业务相关的的需要，而ZK相关的实现由Curator来解决。