---
layout: post
category: "Web Service"
title:  "Java调用WebService（axis2）两种方法 "
tags: [Web Service,Axis2]
---
##方式一：生成客户端代码调用方式。

通过插件工具生成客户端代码进行调用。例如：存在一服务为 http://127.0.0.1：8000/axis2/services/SMSSendService?wsdl通过插件可以生成SMSSendServiceStub.java和SMSSendServiceCallbackHandler.java类。调用的客户端代码如：
	
	 try {
	   SMSSendServiceStub stub=new SMSSendServiceStub();
	   SMSSendServiceStub.method1 m1=new SMSSendServiceStub.method1();

	   m1.setParam1("xxx");
	   try {
	       String ret=stub.multiSend(m1).get_return();
	       System.out.print(ret);
	   } catch (RemoteException e) {
	       e.printStackTrace();
	   }
	  } catch (AxisFault e) {
	   // TODO Auto-generated catch block
	   e.printStackTrace();
	  }

 

##方式二：使用axis2.rpc.client.RPCServiceClient方式调用。

pom.xml配置文件

		<dependency>
			<groupId>org.apache.axis2</groupId>
			<artifactId>axis2</artifactId>
			<version>1.6.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.ws.commons.axiom</groupId>
			<artifactId>axiom-api</artifactId>
			<version>1.2.14</version>
		</dependency>
		<dependency>
			<groupId>org.apache.ws.commons.axiom</groupId>
			<artifactId>axiom-impl</artifactId>
			<version>1.2.14</version>
		</dependency>
		<dependency>
			<groupId>org.apache.xmlbeans</groupId>
			<artifactId>xmlbeans</artifactId>
			<version>2.6.0</version>
		</dependency>
		<dependency>
			<groupId>org.apache.axis2</groupId>
			<artifactId>axis2-transport-http</artifactId>
			<version>1.6.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.axis2</groupId>
			<artifactId>axis2-transport-local</artifactId>
			<version>1.6.2</version>
		</dependency>

调用的代码简单举例如下：
	
	import javax.xml.namespace.QName;

	import org.apache.axis2.addressing.EndpointReference;
	import org.apache.axis2.client.Options;
	import org.apache.axis2.rpc.client.RPCServiceClient;
	
	
	......
	
	try {
	   final String endpoint = "http://127.0.0.1：8000/axis2/services/SMSSendService";
	   String opName = "method1";
	
	   String param="xxx";
	   Object[] opArgs = new Object[] { param };
	
	   Class<?>[] opReturnType = new Class[] { String[].class };
	
	   RPCServiceClient serviceClient = new RPCServiceClient();//此处RPCServiceClient 对象实例建议定义成类中的static变量，否则多次调用会出现连接超时的错误。
	   Options options = serviceClient.getOptions();
	   EndpointReference targetEPR = new EndpointReference(endpoint);
	   options.setTo(targetEPR);
	
	   QName opQName = new QName("http://service.ws.sms.ipcc.ydtf.com",opName);
	   Object[] ret = serviceClient.invokeBlocking(opQName, opArgs,opReturnType);
	   System.out.println(((String[]) ret[0])[0]);
	
	  } catch (AxisFault e) {
	     e.printStackTrace();
	  }
