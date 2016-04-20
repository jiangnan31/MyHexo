title: "Java 多线程<二> 创建与启动(转载)"
date: 2014-10-29 20:58:29
categories: 转载
tags: Concurrency
---
## 一、定义线程
 
### 1、扩展java.lang.Thread类。
 
此类中有个run()方法，应该注意其用法：
```java
public void run()
```
如果该线程是使用独立的 Runnable 运行对象构造的，启动该线程则调用该 Runnable 对象的 run 方法；否则，该方法不执行任何操作并返回。（<span style="color:red">怎么样算使用非独立的 Runnable 运行对象构造？</span>）
<!--more-->
Thread 的子类应该重写该方法。
### 2、实现java.lang.Runnable接口。
 
void run()
使用实现接口 Runnable 的对象创建一个线程时，启动该线程将导致在独立执行的线程中调用对象的 run 方法。
 
方法 run 的常规协定是，它可能执行任何所需的操作。
 
## 二、实例化线程
 
### 1、如果是扩展java.lang.Thread类的线程，则直接new即可。
 
### 2、如果是实现了java.lang.Runnable接口的类，则<span style="color:red">用Thread的构造方法</span>：
Thread(Runnable target) 
Thread(Runnable target, String name) 
Thread(ThreadGroup group, Runnable target) 
Thread(ThreadGroup group, Runnable target, String name) 
Thread(ThreadGroup group, Runnable target, String name, long stackSize)
 
## <span style="color:red">三、启动线程</span>
 
在线程的Thread对象上调用start()方法，而不是run()或者别的方法。
 
在调用start()方法之前：<span style="color:red">线程处于新状态中，新状态指有一个Thread对象，但还没有一个真正的线程。</span>
 
在调用start()方法之后：发生了一系列复杂的事情
```flow
op=>operation: 启动新的执行线程
op1=>operation: 该线程从新状态转移到可运行状态
op2=>operation:  当该线程获得机会执行时，其目标run()方法将运行
op->op1->op2
```

注：启动新的执行线程（具有<span style="color:red">新的调用栈</span>）

注意：对Java来说，run()方法没有任何特别之处。像main()方法一样，它只是新线程知道调用的方法名称(和签名)。因此，在Runnable上或者Thread上调用run方法是合法的。但<span style="color:red">并不启动新的线程</span>。

## 四、实例
```java
package com.test6.Thread;
class TestThread1 implements Runnable {
	private String name;
	 public TestThread1(String name) { 
	        this.name = name;
	    } 
	public void run() {
		for (int i = 1; i <= 50; i++) {
			try {
				Thread.sleep(100);
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
			System.out.println(name+"   :" + i);
		}
	}
}
class TestThread2 extends Thread {
	 public TestThread2(String name) { 
	        super(name); 
	    } 
	public void run() {
		for (int i = 1; i <= 50; i++) {
			try {
				Thread.sleep(150);
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
			System.out.println(this.getName()+"  :" + i);
		}
	}
}
public class ThreadRunnableTest {

	public static void main(String[] args){
	    System.out.println("主线程信息："+Thread.currentThread().getId() +"  "+ Thread.currentThread().getName());
		Runnable  t1 = new TestThread1("thread1");
		Thread thread1 = new Thread(t1);
		thread1.start();
		
		Runnable  t2 = new TestThread2("thread2");
		Thread thread2 = new Thread(t2);
		thread2.start();
	}
}
```
## 五、一些常见问题
 
1、线程的名字，一个运行中的线程总是有名字的，名字有两个来源，一个是虚拟机自己给的名字，一个是你自己的定的名字。在没有指定线程名字的情况下，虚拟机总会为线程指定名字，并且主线程的名字总是main，非主线程的名字不确定。
2、线程都可以设置名字，也可以获取线程的名字，连主线程也不例外。
3、获取当前线程的对象的方法是：Thread.currentThread()；
4、在上面的代码中，只能保证：每个线程都将启动，每个线程都将运行直到完成。一系列线程以某种顺序启动并不意味着将按该顺序执行。对于任何一组启动的线程来说，调度程序不能保证其执行次序，持续时间也无法保证。
5、<span style="color:red">当线程目标run()方法结束时该线程完成。</span>
6、<span style="color:red">一旦线程启动，它就永远不能再重新启动。只有一个新的线程可以被启动，并且只能一次。一个可运行的线程或死线程可以被重新启动。</span>
7、线程的调度是JVM的一部分，在一个CPU的机器上上，实际上一次只能运行一个线程。一次只有一个线程栈执行。JVM线程调度程序决定实际运行哪个处于可运行状态的线程。
众多可运行线程中的某一个会被选中做为当前线程。可运行线程被选择运行的顺序是没有保障的。
8、尽管通常采用队列形式，但这是没有保障的。队列形式是指当一个线程完成“一轮”时，它移到可运行队列的尾部等待，直到它最终排队到该队列的前端为止，它才能被再次选中。事实上，我们把它称为<span style="color:red">可运行池</span>而不是一个可运行队列，目的是帮助认识线程并不都是以某种有保障的顺序排列成一个队列的事实。
9、尽管我们没有无法控制线程调度程序，但可以通过别的方式来影响线程调度的方式。

转载自：[http://lavasoft.blog.51cto.com/62575/27069/](http://lavasoft.blog.51cto.com/62575/27069/)