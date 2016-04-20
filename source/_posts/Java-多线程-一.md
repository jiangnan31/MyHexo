title: "Java 多线程<一> 概念与原理(转载)"
date: 2014-10-28 18:11:13
categories: 转载
tags: Concurrency
---
## 一、操作系统中线程和进程的概念

现在的操作系统是多任务操作系统。多线程是实现多任务的一种方式。

进程是指一个内存中运行的应用程序，每个进程都有自己独立的一块内存空间，一个进程中可以启动多个线程。比如在Windows系统中，一个运行的exe就是一个进程。
 
线程是指进程中的一个执行流程，一个进程中可以运行多个线程。比如java.exe进程中可以运行很多线程。线程总是属于某个进程，进程中的多个线程共享进程的内存。“同时”执行是人的感觉，在线程之间实际上轮换执行。
<!--more-->
多线程的目的是为了<span style="color:red">最大限度的利用CPU资源。</span>

Java编写程序都运行在在Java虚拟机（JVM）中，在JVM的内部，程序的多任务是通过线程来实现的。每用java命令启动一个java应用程序，就会启动一个JVM进程。在同一个JVM进程中，有且只有一个进程，就是它自己。在这个JVM环境中，所有程序代码的运行都是以线程来运行。

一般常见的Java应用程序都是单线程的。<span style="color:red">比如，用java命令运行一个最简单的HelloWorld的Java应用程序时，就启动了一个JVM进程，JVM找到程序程序的入口点main()，然后运行main()方法，这样就产生了一个线程，这个线程称之为主线程。当main方法结束后，主线程运行完成。JVM进程也随即退出。</span>

实际上，操作的系统的多进程实现了多任务并发执行，程序的多线程实现了进程的并发执行(？)。多任务、多进程、多线程的前提都是要求操作系统提供多任务、多进程、多线程的支持。
 
## 二、Java中的线程

在Java中，“线程”指两件不同的事情：
1、java.lang.Thread类的一个实例。
2、线程的执行。
 
使用java.lang.Thread类或者java.lang.Runnable接口编写代码来定义、实例化和启动新线程。
 
一个Thread类实例只是一个对象，像Java中的任何其他对象一样，具有变量和方法，生死于堆上。
 
Java中，每个线程都有一个调用栈，即使不在程序中创建任何新的线程，线程也在后台运行着。
 
一个Java应用总是从main()方法开始运行，mian()方法运行在一个线程内，它被称为主线程。
 
<span style="color:red">一旦创建一个新的线程，就产生一个新的调用栈。</span>
 
线程总体分两类：用户线程和守候线程。

当所有用户线程执行完毕的时候，JVM自动关闭。但是守候线程却不独立于JVM，守候线程一般是由操作系统或者用户自己创建的。（两者唯一区别: 如果User Thread全部撤离，那么Daemon Thread也就没啥线程好服务的了，所以虚拟机也就退出了）。守护线程&非守护线程参考： [这里](http://www.cnblogs.com/super-d2/p/3348183.html)
例子
```java
package com.test5.DaemonThread;

class TestThread1 extends Thread {
	public void run() {
		for (int i = 1; i <= 50; i++) {
			try {
				Thread.sleep(100);
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
			System.out.println("TestThread1:" + i);
		}
	}
}
class TestThread2 extends Thread {
	public void run() {
		for (int i = 1; i <= 50; i++) {
			try {
				Thread.sleep(1000);
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
			System.out.println("TestThread2:" + i);
		}
	}
}
public class TestDemo2  {
	public static void main(String[] args) {
		TestThread1 test1 = new TestThread1();
		TestThread2 test2 = new TestThread2();
		test1.setDaemon(false);// 用户线程,必须结束后才能退出JVM
		test2.setDaemon(true);// 守护线程，不需要结束
		System.out.println(" test1  isDaemon:" + test1.isDaemon());
		System.out.println(" test2  isDaemon:" + test2.isDaemon());
		test1.start();
		test2.start();
		Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
			public void run() {
				System.out.println("jvm运行结束");
			}
		}));
	}
}
```
运行结果：
![](/img/thread/1.png)
从结果可以看出，jvm不必等守护线程执行完成再退出，用户线程执行完毕便可以自动关闭。
附：
Runtime.getRuntime().addShutdownHook(shutdownHook); 
   这个方法的含义说明： 
    这个方法的意思就是在jvm中增加一个关闭的钩子，当jvm关闭的时候，会执行系统中已经设置的所有通过方法addShutdownHook添加的钩子，当系统执行完这些钩子后，jvm才会关闭。所以这些钩子可以在jvm关闭的时候进行内存清理、对象销毁等操作。 
转载自：[http://lavasoft.blog.51cto.com/62575/27069/](http://lavasoft.blog.51cto.com/62575/27069/)