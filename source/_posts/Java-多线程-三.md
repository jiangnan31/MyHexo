title: "Java 多线程<三> 线程栈模型与线程的变量&线程状态的转换(转载)"
date: 2014-10-30 10:09:36
categories: 转载
tags: Concurrency
---
# 一、 线程栈模型与线程的变量

要理解线程调度的原理，以及线程执行过程，必须理解线程栈模型。<span style="color:red">(线程调用start()方法之后会启动新的执行线程，他具有新的调用栈)</span>
线程栈是指某时刻时内存中线程调度的栈信息，<span style="color:red">当前调用的方法总是位于栈顶</span>。线程栈的内容是随着程序的运行动态变化的，因此研究线程栈必须选择一个运行的时刻（实际上指代码运行到什么地方)。
 
下面通过一个示例性的代码说明线程（调用）栈的变化过程。
<!--more-->
![](/img/thread/3_1.png)
 
这幅图描述在代码执行到两个不同时刻1、2时候，虚拟机线程调用栈示意图。
 
当程序执行到t.start();时候，程序多出一个分支（增加了一个调用栈B），这样，栈A、栈B并行执行。
 
从这里就可以看出方法调用和线程启动的区别了。

# 二、 线程状态的转换
## 一、线程状态
线程的状态转换是线程控制的基础。线程状态总的可分为五大状态：分别是生、死、可运行、运行、等待/阻塞。用一个图来描述如下：
![](/img/thread/3_2.png)
1、新状态：线程对象已经创建，还没有在其上调用start()方法。
 
2、可运行状态：当线程有资格运行，但调度程序还没有把它选定为运行线程时线程所处的状态。<span style="color:red">当start()方法调用时，线程首先进入可运行状态。</span>在线程运行之后或者从阻塞、等待或睡眠状态回来后，也返回到可运行状态。
 
3、运行状态：<span style="color:red">线程调度程序</span>从可运行池中选择一个线程作为运行线程时，它所处的状态。这也是线程进入运行状态的唯一一种方式。
 
4、等待/阻塞/睡眠状态：这是线程有资格运行时它所处的状态。实际上这个三状态组合为一种，其共同点是：线程仍旧是活的，但是当前没有条件运行。换句话说，它是可运行的，但是如果某件事件出现，他可能返回到可运行状态。
 
5、死亡态：当线程的run()方法完成时就认为它死去。这个线程对象也许是活的，但是，它已经不是一个单独执行的线程。线程一旦死亡，就不能复生。 如果在一个死去的线程上调用start()方法，会抛出java.lang.IllegalThreadStateException异常。
 
有关详细状态转换图：
线程在一定条件下，状态会发生变化。线程变化的状态转换图如下：
![](/img/thread/3_3.png)
1、新建状态（New）：新创建了一个线程对象。
2、就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3、运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4、阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。
阻塞的情况分三种：
（一）、<span style="color:red">等待阻塞</span>：运行的线程执行<span style="color:red">wait()</span>方法，JVM会把该线程放入等待池中。
（二）、<span style="color:red">同步阻塞</span>：运行的线程在获取对象的同步锁时，若该<span style="color:red">同步锁</span>被别的线程占用，则JVM会把该线程放入锁池中。
（三）、<span style="color:red">其他阻塞</span>：运行的线程<span style="color:red">执行sleep()或join()方法，或者发出了I/O请求时</span>，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。
5、死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。
## 二、阻止线程执行
对于线程的阻止，考虑一下三个方面，不考虑IO阻塞的情况：
睡眠；
等待；
因为需要一个对象的锁定而被阻塞。
### 1、睡眠
Thread.sleep(long millis)和Thread.sleep(long millis, int nanos)静态方法强制当前正在执行的线程休眠（暂停执行），以“减慢线程”。当线程睡眠时，它入睡在某个地方，在苏醒之前不会返回到可运行状态。<span style="color:red">当睡眠时间到期，则返回到可运行状态。</span>
 
线程睡眠的原因：线程执行太快，或者需要强制进入下一轮，因为Java规范不保证合理的轮换。
睡眠的实现：调用静态方法。
```java
        try {
            Thread.sleep(123);
        } catch (InterruptedException e) {
            e.printStackTrace();  
        }
``` 
睡眠的位置：为了让其他线程有机会执行，可以将Thread.sleep()的调用放线程run()之内。这样才能保证该线程执行过程中会睡眠。

我们可以将一个耗时的操作改为睡眠，以减慢线程的执行。这样，线程在每次执行过程中，总会睡眠3毫秒，睡眠了，其他的线程就有机会执行了。
注意：
1、<span style="color:red">线程睡眠是帮助所有线程获得运行机会的最好方法。</span>
2、线程睡眠到期自动苏醒，并返回到<span style="color:red">可运行状态</span>，不是运行状态。sleep()中指定的时间是线程不会运行的最短时间。因此，sleep()方法不能保证该线程睡眠到期后就开始执行。
3、<span style="color:red">sleep()是静态方法，只能控制当前正在运行的线程。</span>
###2、线程的优先级和线程让步yield()
线程的让步是通过Thread.yield()来实现的。<span style="color:red">yield()</span>方法的作用是：<span style="color:red">暂停当前正在执行的线程对象，并执行其他线程。</span>
 
要理解yield()，必须了解线程的优先级的概念。线程总是存在优先级，优先级范围在1~10之间。JVM线程调度程序是基于优先级的抢先调度机制。在大多数情况下，当前运行的线程优先级将大于或等于线程池中任何线程的优先级。但这仅仅是大多数情况。
 
<span style="color:red">注意</span>：当设计多线程应用程序的时候，一定不要依赖于线程的优先级。因为线程调度优先级操作是没有保障的，只能把线程优先级作用作为一种提高程序效率的方法，但是要保证程序不依赖这种操作。
 
当线程池中线程都具有相同的优先级，调度程序的JVM实现自由选择它喜欢的线程。这时候调度程序的操作有两种可能：一是<span style="color:red">选择一个线程运行，直到它阻塞或者运行完成为止。</span>二是<span style="color:red">时间分片，为池内的每个线程提供均等的运行机会。</span>
 
设置线程的优先级：线程默认的优先级是创建它的执行线程的优先级。可以通过setPriority(int newPriority)更改线程的优先级。例如：
        Thread t = new MyThread();
        t.setPriority(8);
        t.start();
线程优先级为1~10之间的正整数，JVM从不会改变一个线程的优先级。然而，1~10之间的值是没有保证的。一些JVM可能不能识别10个不同的值，而将这些优先级进行每两个或多个合并，变成少于10个的优先级，则两个或多个优先级的线程可能被映射为一个优先级。
 
线程默认优先级是5，Thread类中有三个常量，定义线程优先级范围：
static int MAX_PRIORITY         线程可以具有的最高优先级。 
static int MIN_PRIORITY         线程可以具有的最低优先级。 
static int NORM_PRIORITY        分配给线程的默认优先级。
### 3、Thread.yield()方法详解
 
Thread.yield()方法作用是：暂停当前正在执行的线程对象，并执行其他线程。
yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。
结论：yield()从未导致线程转到等待/睡眠/阻塞状态。<span style="color:red">在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。</spn>
 
### 4、join()方法
 在JDk的API里对于join()方法是：

join
public final void join() throws InterruptedException Waits for this thread to die. Throws: InterruptedException  - if any thread has interrupted the current thread. The interrupted status of the current thread is cleared when this exception is thrown.

<span style="color:red">即join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。</spn>

Thread的非静态方法join()让一个主线程B“加入”到另外一个子线程A的尾部。在A执行完毕之前，B不能工作。例如：
```java
        Thread A = new MyThread();
        A.start();
        A.join();
		...
```
说明： [这里](http://www.open-open.com/lib/view/open1371741636171.html) [这里](http://www.blogjava.net/jnbzwm/articles/330549.html)
另外，join()方法还有带超时限制的重载版本。 例如t.join(5000);则让线程等待5000毫秒，如果超过这个时间，则停止等待，变为可运行状态。
 
线程的加入join()对线程栈导致的结果是<span style="color:red">线程栈</span>发生了变化，当然这些变化都是瞬时的。

![](/img/thread/3_4.png)

代码说明：
```java
package com.test6.Thread;
class MyThread extends Thread {
	 public MyThread(String name) { 
	        super(name); 
	    } 
	public void run() {
		for (int i = 1; i <= 10; i++) {
			try {
				Thread.sleep(150);
			} catch (InterruptedException ex) {
				ex.printStackTrace();
			}
			System.out.println(this.getName()+"  :" + i);
		}
	}
}
public class TestJoin {
	public static void main(String[] args) throws InterruptedException{
		MyThread t1 = new MyThread("thread1");
		MyThread t2 = new MyThread("thread2");
		t1.start();
		//t1.join();
		t2.start();
		Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
			public void run() {
				System.out.println("jvm运行结束");
			}
		}));
	}
}
```
运行结果
![](/img/thread/3_7.png)

小结
到目前位置，介绍了线程离开运行状态的3种方法：
1、调用Thread.sleep()：使当前线程睡眠至少多少毫秒（尽管它可能在指定的时间之前被中断）。
2、调用Thread.yield()：不能保障太多事情，尽管通常它会让当前运行线程回到可运行性状态，使得有相同优先级的线程有机会执行。
3、调用join()方法：保证当前线程停止执行，直到该线程所加入的线程完成为止。然而，如果它加入的线程没有存活，则当前线程不需要停止。
 
除了以上三种方式外，还有下面几种特殊情况可能使线程离开运行状态：
1、线程的run()方法完成。
2、在对象上调用wait()方法（不是在线程上调用）。
3、线程不能在对象上获得锁定，它正试图运行该对象的方法代码。
4、线程调度程序可以决定将当前运行状态移动到可运行状态，以便让另一个线程获得运行机会，而不需要任何理由。
转载自：[http://lavasoft.blog.51cto.com/62575/27069/](http://lavasoft.blog.51cto.com/62575/27069/)