---
layout: post
category: "HttpClient"
title:  "HttpClient及有关jar包详解"
tags: [HttpClient]
---

##1.HttpClient简介
HttpClient 是 Apache Jakarta Common 下的子项目，可以用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。

本文首先介绍 HTTPClient，然后根据作者实际工作经验给出了一些常见问题的解决方法。HTTP 协议可能是现在 Internet 上使用得最多、最重要的协议了，越来越多的 Java 应用程序需要直接通过 HTTP 协议来访问网络资源。

虽然在 JDK 的 java.net 包中已经提供了访问 HTTP 协议的基本功能，但是对于大部分应用程序来说，JDK 库本身提供的功能还不够丰富和灵活。HttpClient 是 Apache Jakarta Common 下的子项目，用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。HttpClient 已经应用在很多的项目中，比如 Apache Jakarta 上很著名的另外两个开源项目 Cactus 和 HTMLUnit 都使用了 HttpClient。现在HttpClient最新版本为 HttpClient 4.0-beta2
##2.HttpClient 功能介绍
	以下列出的是 HttpClient 提供的主要的功能，要知道更多详细的功能可以参见 HttpClient 的主页。
	（1）实现了所有 HTTP 的方法（GET,POST,PUT,HEAD 等）
	（2）支持自动转向
	（3）支持 HTTPS 协议
	（4）支持代理服务器等
##3.HttpClient 基本功能的使用
###(1) GET方法
	使用 HttpClient 需要以下 6 个步骤：
	1. 创建 HttpClient 的实例
	2. 创建某种连接方法的实例，在这里是 GetMethod。在 GetMethod 的构造函数中传入待连接的地址
	3. 调用第一步中创建好的实例的 execute 方法来执行第二步中创建好的 method 实例
	4. 读 response
	5. 释放连接。无论执行方法是否成功，都必须释放连接
	6. 对得到后的内容进行处理
根据以上步骤，我们来编写用GET方法来取得某网页内容的代码。
大部分情况下 HttpClient 默认的构造函数已经足够使用。 HttpClient httpClient = new HttpClient();
创建GET方法的实例。在GET方法的构造函数中传入待连接的地址即可。用GetMethod将会自动处理转发过程，如果想要把自动处理转发过程去掉的话，可以调用方法setFollowRedirects(false)。 GetMethod getMethod = new GetMethod(".....");
调用实例httpClient的executeMethod方法来执行getMethod。由于是执行在网络上的程序，在运行executeMethod方法的时候，需要处理两个异常，分别是HttpException和IOException。引起第一种异常的原因主要可能是在构造getMethod的时候传入的协议不对，比如不小心将"http"写成"htp"，或者服务器端返回的内容不正常等，并且该异常发生是不可恢复的；第二种异常一般是由于网络原因引起的异常，对于这种异常 （IOException），HttpClient会根据你指定的恢复策略自动试着重新执行executeMethod方法。HttpClient的恢复策略可以自定义（通过实现接口HttpMethodRetryHandler来实现）。通过httpClient的方法setParameter设置你实现的恢复策略，本文中使用的是系统提供的默认恢复策略，该策略在碰到第二类异常的时候将自动重试3次。executeMethod返回值是一个整数，表示了执行该方法后服务器返回的状态码，该状态码能表示出该方法执行是否成功、需要认证或者页面发生了跳转（默认状态下GetMethod的实例是自动处理跳转的）等。 //设置成了默认的恢复策略，在发生异常时候将自动重试3次，在这里你也可以设置成自定义的恢复策略
getMethod.getParams().setParameter(HttpMethodParams.RETRY_HANDLER,
new DefaultHttpMethodRetryHandler());
//执行getMethod
int statusCode = client.executeMethod(getMethod);
if (statusCode != HttpStatus.SC_OK) {
System.err.println("Method failed: " + getMethod.getStatusLine());
}
在返回的状态码正确后，即可取得内容。取得目标地址的内容有三种方法：第一种，getResponseBody，该方法返回的是目标的二进制的byte流；第二种，getResponseBodyAsString，这个方法返回的是String类型，值得注意的是该方法返回的String的编码是根据系统默认的编码方式，所以返回的String值可能编码类型有误，在本文的"字符编码"部分中将对此做详细介绍；第三种，getResponseBodyAsStream，这个方法对于目标地址中有大量数据需要传输是最佳的。在这里我们使用了最简单的getResponseBody方法。 byte[] responseBody = method.getResponseBody();
释放连接。无论执行方法是否成功，都必须释放连接。 method.releaseConnection();
处理内容。在这一步中根据你的需要处理内容，在例子中只是简单的将内容打印到控制台。 System.out.println(new String(responseBody));
下面是程序的完整代码，这些代码也可在附件中的test.GetSample中找到。
package test;
import java.io.IOException;
import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.GetMethod;
import org.apache.commons.httpclient.params.HttpMethodParams;
public class GetSample{
public static void main(String[] args) {
//构造HttpClient的实例
HttpClient httpClient = new HttpClient();
//创建GET方法的实例
GetMethod getMethod = new GetMethod("...");
//使用系统提供的默认的恢复策略
getMethod.getParams().setParameter(HttpMethodParams.RETRY_HANDLER,
new DefaultHttpMethodRetryHandler());
try {
//执行getMethod
int statusCode = httpClient.executeMethod(getMethod);
if (statusCode != HttpStatus.SC_OK) {
System.err.println("Method failed: "
+ getMethod.getStatusLine());
}
//读取内容
byte[] responseBody = getMethod.getResponseBody();
//处理内容
System.out.println(new String(responseBody));
} catch (HttpException e) {
//发生致命的异常，可能是协议不对或者返回的内容有问题
System.out.println("Please check your provided http address!");
e.printStackTrace();
} catch (IOException e) {
//发生网络异常
e.printStackTrace();
} finally {
//释放连接
getMethod.releaseConnection();
}
}
}
（2）POST方法
根据RFC2616，对POST的解释如下：POST方法用来向目的服务器发出请求，要求它接受被附在请求后的实体，并把它当作请求队列（Request-Line）中请求URI所指定资源的附加新子项。POST被设计成用统一的方法实现下列功能：
对现有资源的注释（Annotation of existing resources）
向电子公告栏、新闻组，邮件列表或类似讨论组发送消息
提交数据块，如将表单的结果提交给数据处理过程
通过附加操作来扩展数据库
调用HttpClient中的PostMethod与GetMethod类似，除了设置PostMethod的实例与GetMethod有些不同之外，剩下的步骤都差不多。在下面的例子中，省去了与GetMethod相同的步骤，只说明与上面不同的地方，并以登录清华大学BBS为例子进行说明。
构造PostMethod之前的步骤都相同，与GetMethod一样，构造PostMethod也需要一个URI参数。在创建了PostMethod的实例之后，需要给method实例填充表单的值，在BBS的登录表单中需要有两个域，第一个是用户名（域名叫id），第二个是密码（域名叫passwd）。表单中的域用类NameValuePair来表示，该类的构造函数第一个参数是域名，第二参数是该域的值；将表单所有的值设置到PostMethod中用方法setRequestBody。另外由于BBS登录成功后会转向另外一个页面，但是HttpClient对于要求接受后继服务的请求，比如POST和PUT，不支持自动转发，因此需要自己对页面转向做处理。具体的页面转向处理请参见下面的"自动转向"部分。代码如下：
String url = "....";
PostMethod postMethod = new PostMethod(url);
// 填入各个表单域的值
NameValuePair[] data = { new NameValuePair("id", "youUserName"),
new NameValuePair("passwd", "yourPwd") };
// 将表单的值放入postMethod中
postMethod.setRequestBody(data);
// 执行postMethod
int statusCode = httpClient.executeMethod(postMethod);
// HttpClient对于要求接受后继服务的请求，象POST和PUT等不能自动处理转发
// 301或者302
if (statusCode == HttpStatus.SC_MOVED_PERMANENTLY ||
statusCode == HttpStatus.SC_MOVED_TEMPORARILY) {
// 从头中取出转向的地址
Header locationHeader = postMethod.getResponseHeader("location");
String location = null;
if (locationHeader != null) {
location = locationHeader.getValue();
System.out.println("The page was redirected to:" + location);
} else {
System.err.println("Location field value is null.");
}
return;
}
[编辑本段]
4 使用HttpClient过程中常见的一些问题
下面介绍在使用HttpClient过程中常见的一些问题。
字符编码
某目标页的编码可能出现在两个地方，第一个地方是服务器返回的http头中，另外一个地方是得到的html/xml页面中。
在http头的Content-Type字段可能会包含字符编码信息。例如可能返回的头会包含这样子的信息：Content-Type: text/html; charset=UTF-8。这个头信息表明该页的编码是UTF-8，但是服务器返回的头信息未必与内容能匹配上。比如对于一些双字节语言国家，可能服务器返回的编码类型是UTF-8，但真正的内容却不是UTF-8编码的，因此需要在另外的地方去得到页面的编码信息；但是如果服务器返回的编码不是UTF-8，而是具体的一些编码，比如gb2312等，那服务器返回的可能是正确的编码信息。通过method对象的getResponseCharSet()方法就可以得到http头中的编码信息。
对于象xml或者html这样的文件，允许作者在页面中直接指定编码类型。比如在html中会有<meta http-equiv="Content-Type" content="text/html; charset=gb2312"/>这样的标签；或者在xml中会有<?xml version="1.0" encoding="gb2312"?>这样的标签，在这些情况下，可能与http头中返回的编码信息冲突，需要用户自己判断到底那种编码类型应该是真正的编码。
自动转向
根据RFC2616中对自动转向的定义，主要有两种：301和302。301表示永久的移走（Moved Permanently），当返回的是301，则表示请求的资源已经被移到一个固定的新地方，任何向该地址发起请求都会被转到新的地址上。302表示暂时的转向，比如在服务器端的servlet程序调用了sendRedirect方法，则在客户端就会得到一个302的代码，这时服务器返回的头信息中location的值就是sendRedirect转向的目标地址。
HttpClient支持自动转向处理，但是象POST和PUT方式这种要求接受后继服务的请求方式，暂时不支持自动转向，因此如果碰到POST方式提交后返回的是301或者302的话需要自己处理。就像刚才在POSTMethod中举的例子：如果想进入登录BBS后的页面，必须重新发起登录的请求，请求的地址可以在头字段location中得到。不过需要注意的是，有时候location返回的可能是相对路径，因此需要对location返回的值做一些处理才可以发起向新地址的请求。
另外除了在头中包含的信息可能使页面发生重定向外，在页面中也有可能会发生页面的重定向。引起页面自动转发的标签是：<meta http-equiv="refresh" content="5; url=....">。如果你想在程序中也处理这种情况的话得自己分析页面来实现转向。需要注意的是，在上面那个标签中url的值也可以是一个相对地址，如果是这样的话，需要对它做一些处理后才可以转发。
处理HTTPS协议
HttpClient提供了对SSL的支持，在使用SSL之前必须安装JSSE。在Sun提供的1.4以后的版本中，JSSE已经集成到JDK中，如果你使用的是JDK1.4以前的版本则必须安装JSSE。JSSE不同的厂家有不同的实现。下面介绍怎么使用HttpClient来打开Https连接。这里有两种方法可以打开https连接，第一种就是得到服务器颁发的证书，然后导入到本地的keystore中；另外一种办法就是通过扩展HttpClient的类来实现自动接受证书。
方法1，取得证书，并导入本地的keystore：
安装JSSE （如果你使用的JDK版本是1.4或者1.4以上就可以跳过这一步）。本文以IBM的JSSE为例子说明。先到IBM网站上下载JSSE的安装包。然后解压开之后将ibmjsse.jar包拷贝到<java-home>\lib\ext\目录下。
取得并且导入证书。证书可以通过IE来获得：
1． 用IE打开需要连接的https网址，会弹出如下对话框：
2． 单击"View Certificate"，在弹出的对话框中选择"Details"，然后再单击"Copy to File"，根据提供的向导生成待访问网页的证书文件
3． 向导第一步，欢迎界面，直接单击"Next"，
4． 向导第二步，选择导出的文件格式，默认，单击"Next"，
5． 向导第三步，输入导出的文件名，输入后，单击"Next"，
6． 向导第四步，单击"Finish"，完成向导
7． 最后弹出一个对话框，显示导出成功
用keytool工具把刚才导出的证书倒入本地keystore。Keytool命令在<java-home>\bin\下，打开命令行窗口，并到<java-home>\lib\security\目录下，运行下面的命令：
keytool -import -noprompt -keystore cacerts -storepass changeit -alias yourEntry1 -file your.cer
其中参数alias后跟的值是当前证书在keystore中的唯一标识符，但是大小写不区分；参数file后跟的是刚才通过IE导出的证书所在的路径和文件名；如果你想删除刚才导入到keystore的证书，可以用命令：
keytool -delete -keystore cacerts -storepass changeit -alias yourEntry1
写程序访问https地址。如果想测试是否能连上https，只需要稍改一下GetSample例子，把请求的目标变成一个https地址。
GetMethod getMethod = new GetMethod("your url");
运行该程序可能出现的问题：
1. 抛出异常java.net.SocketException: Algorithm SSL not available。出现这个异常可能是因为没有加JSSEProvider，如果用的是IBM的JSSE Provider，在程序中加入这样的一行：
if(Security.getProvider("com.ibm.jsse.IBMJSSEProvider") == null)
Security.addProvider(new IBMJSSEProvider());
或者也可以打开<java-home>\lib\security\java.security，在行
security.provider.1=sun.security.provider.Sun
security.provider.2=com.ibm.crypto.provider.IBMJCE
后面加入security.provider.3=com.ibm.jsse.IBMJSSEProvider
2. 抛出异常java.net.SocketException: SSL implementation not available。出现这个异常可能是你没有把ibmjsse.jar拷贝到<java-home>\lib\ext\目录下。
3. 抛出异常javax.net.ssl.SSLHandshakeException: unknown certificate。出现这个异常表明你的JSSE应该已经安装正确，但是可能因为你没有把证书导入到当前运行JRE的keystore中，请按照前面介绍的步骤来导入你的证书。
方法２，扩展HttpClient类实现自动接受证书
因为这种方法自动接收所有证书，因此存在一定的安全问题，所以在使用这种方法前请仔细考虑您的系统的安全需求。具体的步骤如下：
提供一个自定义的socket factory（test.MySecureProtocolSocketFactory）。这个自定义的类必须实现接口org.apache.commons.httpclient.protocol.SecureProtocolSocketFactory，在实现接口的类中调用自定义的X509TrustManager(test.MyX509TrustManager)，这两个类可以在随本文带的附件中得到
创建一个org.apache.commons.httpclient.protocol.Protocol的实例，指定协议名称和默认的端口号 Protocol myhttps = new Protocol("https", new MySecureProtocolSocketFactory (), 443);
注册刚才创建的https协议对象 Protocol.registerProtocol("https ", myhttps);
然后按照普通编程方式打开https的目标地址，代码请参见test.NoCertificationHttpsGetSample
[编辑本段]
5 处理代理服务器
HttpClient中使用代理服务器非常简单，调用HttpClient中setProxy方法就可以，方法的第一个参数是代理服务器地址，第二个参数是端口号。另外HttpClient也支持SOCKS代理。
httpClient.getHostConfiguration().setProxy(hostName,port);

原文链接：http://blog.csdn.net/csh624366188/article/details/7183446