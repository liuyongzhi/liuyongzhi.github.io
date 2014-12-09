---
layout: post
category: "Spring MVC"
title:  "Spring MVC Controller 单元测试 - pairwinter"
tags: [Spring MVC,Controller]
---
##一、简介
Controller层的单元测试可以使得应用的可靠性得到提升，虽然这使得开发的时间有所增加，有得必失，这里我认为得到的比失去的多很多。
	
Sping MVC3.2版本之后的单元测试方法有所变化，随着功能的提升，单元测试更加的简单高效。
	
这里以4.1版本为例，记录Controller的单元测试流程。非常值得参考的是Spring MVC Showcase
([https://github.com/spring-projects/spring-mvc-showcase](https://github.com/spring-projects/spring-mvc-showcase "https://github.com/spring-projects/spring-mvc-showcase"))，它当前的版本使用的是4.1.0，
	以后会有所变动，为了使项目能够运行，请以它更新的配置为参考。

##二、项目结构

使用maven构建项目，项目结构如下：

    WEB-INF
        web.xml (文件)
        spring （文件夹）
            root-context.xml （文件，放置能够被servlet和filter共享使用的资源）
            appServlet （目录）
                controllers.xml（文件，与Controller相关的配置）
                servlet-context.xml （文件，放置有servlet使用的资源）

Spring mvc是对Servlet的包装，使其能够结构化，流程化。

	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" version="2.5">
	  <context-param>
	    <param-name>contextConfigLocation</param-name>
	    <param-value>/WEB-INF/spring/root-context.xml</param-value>
	  </context-param>
	  <listener>
	    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	  </listener>
	  <servlet>
	    <servlet-name>appServlet</servlet-name>
	    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	    <init-param>
	      <param-name>contextConfigLocation</param-name>
	      <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
	    </init-param>
	    <load-on-startup>1</load-on-startup>
	  </servlet>
	  <servlet-mapping>
	    <servlet-name>appServlet</servlet-name>
	    <url-pattern>/</url-pattern>
	  </servlet-mapping>
	</web-app>

可以看到分开配置，使得文件的作用更加的明了。

这部分的配置文件是来配置web context 的，项目中还有其他的module ，如DAO，Service，他们对应的applicationContext文件会被放在src/main/resource目录下。

完善的单元测试当然还有service的单元测试，这里就不说了，但是Controller的单元测试还需要调用service和DAO，要注意Service和DAO的applicationContext的引入。

##三、Controller 单元测试

在测试类中包含这三个注释，看起表面意思不难理解他们的作用，主要理解ContextConfiguration的使用。
	
	@RunWith(SpringJUnit4ClassRunner.class)
	@WebAppConfiguration //默认是src/main/webapp
	@ContextConfiguration("file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml")

注意：这里的@ContextConfiguration只解析了servlet-context.xml，如果项目中还存在其他模块的applicationContext，也需要把他们引进来否则得到的Service就是null的。例如：
	
	@ContextConfiguration({
	"file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml",
	“classpath*: springxml/**.xml”
	
	})

在加上其他的一点代码就可以完成一个Controller的单元测试，下面是一个例子，更多例子请参考showcase中的内容。
	
	package pairwinter.spring.mvc.controller.test;

	import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
	import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
	import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
	import static org.springframework.test.web.servlet.setup.MockMvcBuilders.webAppContextSetup;

	import org.junit.Before;
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.samples.mvc.AbstractContextControllerTests;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
	import org.springframework.test.web.servlet.MockMvc;

	@RunWith(SpringJUnit4ClassRunner.class)
	@WebAppConfiguration
	@ContextConfiguration({
		"file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml",
		“classpath*: springxml/**.xml”
	})
	
	public class ControllerTests{
	
		@Autowired
		private WebApplicationContext wac;
	
		private MockMvc mockMvc;
		
		@Before
		public void setup() throws Exception {
			this.mockMvc = webAppContextSetup(this.wac).build();
		}
		
		@Test
		
		public void controllerExceptionHandler() throws Exception {
			this.mockMvc.perform(get("/test"))
			.andExpect(status().isOk());
		}
	}