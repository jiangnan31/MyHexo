title: Thrift java 使用实例(修改版)
date: 2014-10-22 17:18:05
categories: 笔记
tags: RPC
---
## 1. Thirft简介
根据Apache Thrift的官方站点的描述，Thrift是一个：
software framework, for scalable cross-language services development, combines a software stack with a code generation engine to build services that work efficiently and seamlessly between C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, OCaml and Delphi and other languages.
安装Thrift比较的烦，但是在Windows下官方编译了一个thrift.exe，下载就行了。点击一闪即逝 ，木有关系。
拷贝到c:\windows目录下(或者任何目录下)，然后就可以在dos环境下使用了
## 2. 写Thrift定义文件（.thrift file）
示例定义文件（add.thrift）

<!--more-->
```xml
namespace java com.samples.thrift.server  // defines the namespace   
   
typedef i32 int  //typedefs to get convenient names for your types  
   
service AdditionService {  // defines the service to add two numbers  
        int add(1:int n1, 2:int n2), //defines a method  
}
```
## 3. 编译Thrift定义文件）
下面的命令编译.thrift文件
```sh
thrift --gen <language> <Thrift filename>
```
对于我的例子来讲，命令是：
```sh
thrift-0.9.1 -gen java E:\workspace\ThriftTest\src\main\java\add.thrift
```
![](/img/thrift_1.png)
在执行完代码后，在gen-java目录下你会发现构建RPC服务器和客户端有用的源代码。创建一个叫做AddtionService.java的java文件。
## 4. 创建Maven项目
### 4.1 pom.xml中添加所需jar包信息
```xml
<dependencies>
		<dependency>
			<groupId>org.apache.thrift</groupId>
			<artifactId>libthrift</artifactId>
			<version>0.9.1</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.5.7</version>
		</dependency>
	</dependencies>
```
### 4.2 写一个 service handler AdditionServiceHandler.java 
Service handler 类必须实现 AdditionService.Iface接口。
示例Service handler（AdditionServiceHandler.java）
```java
package com.samples.thrift.server;

import org.apache.thrift.TException;  

public class AdditionServiceHandler implements AdditionService.Iface {  
   
 public int add(int n1, int n2) throws TException {  
  return n1 + n2;  
 }  
   
}
```
### 4.3 写一个简单的服务器 MyServer.java
下面的示例代码是一个简单的Thrift服务器。可以看到下面的代码中有一段是注释了的，可以去掉注释来启用多线程服务器。
示例服务器（MyServer.java）
```java
package com.samples.thrift.server;

import org.apache.thrift.transport.TServerSocket;  
import org.apache.thrift.transport.TServerTransport;  
import org.apache.thrift.server.TServer;  
import org.apache.thrift.server.TServer.Args;  
import org.apache.thrift.server.TSimpleServer;  
   
public class MyServer {  
   
 public static void StartsimpleServer(AdditionService.Processor<AdditionServiceHandler> processor) {  
  try {  
   TServerTransport serverTransport = new TServerSocket(4578);  
   TServer server = new TSimpleServer(  
     new Args(serverTransport).processor(processor));  
   
   // Use this for a multithreaded server  
   // TServer server = new TThreadPoolServer(new  
   // TThreadPoolServer.Args(serverTransport).processor(processor));  
   
   System.out.println("Starting the simple server...");  
   server.serve();  
  } catch (Exception e) {  
   e.printStackTrace();  
  }  
 }  
    
 public static void main(String[] args) {  
  StartsimpleServer(new AdditionService.Processor<AdditionServiceHandler>(new AdditionServiceHandler()));  
 }  
   
}
```
### 4.4 写一个客户端  AdditionClient.java 
下面的例子是一个使用Java写的客户端短使用AdditionService的服务。
```java
package com.samples.thrift.server;

import org.apache.thrift.TException;  
import org.apache.thrift.protocol.TBinaryProtocol;  
import org.apache.thrift.protocol.TProtocol;  
import org.apache.thrift.transport.TSocket;  
import org.apache.thrift.transport.TTransport;  
import org.apache.thrift.transport.TTransportException;  
   
public class AdditionClient {  
   
 public static void main(String[] args) {  
   
  try {  
   TTransport transport;  
   
   transport = new TSocket("localhost", 4578);  
   transport.open();  
   
   TProtocol protocol = new TBinaryProtocol(transport);  
   AdditionService.Client client = new AdditionService.Client(protocol);  
   
   System.out.println(client.add(100, 200));  
   
   transport.close();  
  } catch (TTransportException e) {  
   e.printStackTrace();  
  } catch (TException x) {  
   x.printStackTrace();  
  }  
 }  
   
}
```

运行服务端代码（MyServer.java）将会看到下面的输出。

    Starting the simple server...

然后运行客户端代码（AdditionClient.java），将会看到如下输出。

    300
    
---
参考文档：
[参考1](http://my.oschina.net/jack230230/blog/66041)
[参考2](http://www.cnblogs.com/liyonghui/archive/2013/07/12/3186140.html)
[参考3](http://gemantic.iteye.com/blog/1199214)
[参考4](https://thrift.apache.org/)

               
网盘下载：[http://pan.baidu.com/s/1sjI1S4l](http://pan.baidu.com/s/1sjI1S4l)
[http://blog.csdn.net/qiangweiloveforever/article/details/7195145](http://blog.csdn.net/qiangweiloveforever/article/details/7195145)，除此之外，博客还有很多很有深度的文章