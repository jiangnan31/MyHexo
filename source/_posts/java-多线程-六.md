title: "java 多线程<六>"
date: 2014-12-09 20:16:50
categories: 转载
tags: Concurrency
---
# 线程的调度-优先级
 
与线程休眠类似，线程的优先级仍然无法保障线程的执行次序。只不过，优先级高的线程获取CPU资源的概率较大，优先级低的并非没机会执行。
 
线程的优先级用1-10之间的整数表示，数值越大优先级越高，默认的优先级为5。
 
在一个线程中开启另外一个新线程，则新开线程称为该线程的子线程，子线程初始优先级与父线程相同。

<!--more-->
```java
/** 
* Java线程：线程的调度-优先级 
* 
* @author leizhimin 2009-11-4 9:02:40 
*/ 
class MyThread1 extends Thread {
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("线程1第" + i + "次执行！");
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

class MyRunnable implements Runnable {
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("线程2第" + i + "次执行！");
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}

public class TestPriority {
	public static void main(String[] args) {
		Thread t1 = new MyThread1();
		Thread t2 = new Thread(new MyRunnable());
		t1.setPriority(10);
		t2.setPriority(1);

		t2.start();
		t1.start();
	}
}
```
**Result:**
线程2第0次执行！
线程1第0次执行！
线程1第1次执行！
线程2第1次执行！
线程1第2次执行！
线程2第2次执行！
线程1第3次执行！
线程2第3次执行！
线程1第4次执行！
线程2第4次执行！
线程1第5次执行！
线程2第5次执行！
线程1第6次执行！
线程2第6次执行！
线程1第7次执行！
线程2第7次执行！
线程1第8次执行！
线程2第8次执行！
线程1第9次执行！
线程2第9次执行！ 


# 线程的调度-让步
 
线程的让步含义就是使当前运行着线程让出CPU资源，但是然给谁不知道，仅仅是让出，线程状态回到可运行状态。
 
线程的让步使用Thread.yield()方法，yield() 为静态方法，功能是暂停当前正在执行的线程对象，并执行其他线程。
```Java
/** 
* Java线程：线程的调度-让步 
* 
*/ 
class MyThread2 extends Thread { 
    public void run() { 
            for (int i = 0; i < 10; i++) { 
                    System.out.println("线程1第" + i + "次执行！"); 
            } 
    } 
} 

class MyRunnable2 implements Runnable { 
    public void run() { 
            for (int i = 0; i < 10; i++) { 
                    System.out.println("线程2第" + i + "次执行！"); 
                    Thread.yield(); 
            } 
    } 
}

public class TestThreadYield { 
        public static void main(String[] args) { 
                Thread t1 = new MyThread2(); 
                Thread t2 = new Thread(new MyRunnable2()); 

                t2.start(); 
                t1.start(); 
        } 
} 
```
 
线程2第0次执行！
线程2第1次执行！
线程1第0次执行！
线程1第1次执行！
线程1第2次执行！
线程1第3次执行！
线程2第2次执行！
线程1第4次执行！
线程2第3次执行！
线程2第4次执行！
线程2第5次执行！
线程1第5次执行！
线程1第6次执行！
线程2第6次执行！
线程2第7次执行！
线程2第8次执行！
线程1第7次执行！
线程1第8次执行！
线程1第9次执行！
线程2第9次执行！ 

<span style="color:red">yiedld这个方法是让当前线程回到可执行状态，以便让具有相同优先级的线程进入执行状态，但不是绝对的。因为虚拟机可能会让该线程重新进入执行状态。？？</span>


# 线程的调度-合并
 
线程的合并的含义就是将几个并行线程的线程合并为一个单线程执行，应用场景是<span style="color:red">当一个线程必须等待另一个线程执行完毕才能执行时可以使用join方法。</span>
 
join为非静态方法，定义如下：
void join()    
    等待该线程终止。    
void join(long millis)    
    等待该线程终止的时间最长为 millis 毫秒。    
void join(long millis, int nanos)    
    等待该线程终止的时间最长为 millis 毫秒 + nanos 纳秒。
```java
/** 
* Java线程：线程的调度-合并 
* 
* @author leizhimin 2009-11-4 9:02:40 
*/ 
public class Test { 
        public static void main(String[] args) { 
                Thread t1 = new MyThread1(); 
                t1.start(); 

                for (int i = 0; i < 20; i++) { 
                        System.out.println("主线程第" + i + "次执行！"); 
                        if (i > 2) try { 
                                //t1线程合并到主线程中，主线程停止执行过程，转而执行t1线程，直到t1执行完毕后继续。 
                                t1.join(); 
                        } catch (InterruptedException e) { 
                                e.printStackTrace(); 
                        } 
                } 
        } 
} 

class MyThread1 extends Thread { 
        public void run() { 
                for (int i = 0; i < 10; i++) { 
                        System.out.println("线程1第" + i + "次执行！"); 
                } 
        } 
}
```
主线程第0次执行！
线程1第0次执行！
主线程第1次执行！
线程1第1次执行！
主线程第2次执行！
主线程第3次执行！
线程1第2次执行！
线程1第3次执行！
线程1第4次执行！
线程1第5次执行！
线程1第6次执行！
线程1第7次执行！
线程1第8次执行！
线程1第9次执行！
join结束
主线程第4次执行！
join结束
主线程第5次执行！
join结束
主线程第6次执行！
join结束
主线程第7次执行！
join结束
主线程第8次执行！
join结束
主线程第9次执行！
join结束
主线程第10次执行！
join结束
主线程第11次执行！
join结束
主线程第12次执行！
join结束
主线程第13次执行！
join结束
主线程第14次执行！
join结束
主线程第15次执行！
join结束
主线程第16次执行！
join结束
主线程第17次执行！
join结束
主线程第18次执行！
join结束
主线程第19次执行！
join结束

# 线程的调度-守护线程
 
守护线程与普通线程写法上基本么啥区别，调用线程对象的方法setDaemon(true)，则可以将其设置为守护线程。
 
守护线程使用的情况较少，但并非无用，举例来说，JVM的垃圾回收、内存管理等线程都是守护线程。还有就是在做数据库应用时候，使用的数据库连接池，连接池本身也包含着很多后台线程，监控连接个数、超时时间、状态等等。
 
setDaemon方法的详细说明：
public final void setDaemon(boolean on)将该线程标记为守护线程或用户线程。当正在运行的线程都是守护线程时，Java 虚拟机退出。    
  该方法必须在启动线程前调用。


  该方法首先调用该线程的 checkAccess 方法，且不带任何参数。这可能抛出 SecurityException（在当前线程中）。    


  参数： 
    on - 如果为 true，则将该线程标记为守护线程。    
  抛出：    
    IllegalThreadStateException - 如果该线程处于活动状态。    
    SecurityException - 如果当前线程无法修改该线程。 
  另请参见： 
    isDaemon(), checkAccess()
Thread源码：
```java
/**
     * Marks this thread as either a daemon thread or a user thread. The 
     * Java Virtual Machine exits when the only threads running are all 
     * daemon threads. 
     * <p>
     * This method must be called before the thread is started. 
      * <p>
     * This method first calls the <code>checkAccess</code> method 
     * of this thread 
     * with no arguments. This may result in throwing a 
     * <code>SecurityException </code>(in the current thread). 
    *
     * @param      on   if <code>true</code>, marks this thread as a
     *                  daemon thread.
     * @exception  IllegalThreadStateException  if this thread is active.
     * @exception  SecurityException  if the current thread cannot modify
     *               this thread.
     * @see        #isDaemon()
     * @see        #checkAccess
     */
    public final void setDaemon(boolean on) {
	checkAccess();
	if (isAlive()) {
	    throw new IllegalThreadStateException();
	}
	daemon = on;
    }
```
例子：
```java
/** 
* Java线程：线程的调度-守护线程 
* 
* @author leizhimin 2009-11-4 9:02:40 
*/ 
public class Test { 
        public static void main(String[] args) { 
                Thread t1 = new MyCommon(); 
                Thread t2 = new Thread(new MyDaemon()); 
                t2.setDaemon(true);        //设置为守护线程 

                t2.start(); 
                t1.start(); 
        } 
} 

class MyCommon extends Thread { 
        public void run() { 
                for (int i = 0; i < 5; i++) { 
                        System.out.println("线程1第" + i + "次执行！"); 
                        try { 
                                Thread.sleep(7); 
                        } catch (InterruptedException e) { 
                                e.printStackTrace(); 
                        } 
                } 
        } 
} 

class MyDaemon implements Runnable { 
        public void run() { 
                for (long i = 0; i < 9999999L; i++) { 
                        System.out.println("后台线程第" + i + "次执行！"); 
                        try { 
                                Thread.sleep(7); 
                        } catch (InterruptedException e) { 
                                e.printStackTrace(); 
                        } 
                } 
        } 
}
```


**Result:**
后台线程第0次执行！ 
线程1第0次执行！ 
线程1第1次执行！ 
后台线程第1次执行！ 
后台线程第2次执行！ 
线程1第2次执行！ 
线程1第3次执行！ 
后台线程第3次执行！ 
线程1第4次执行！ 
后台线程第4次执行！ 
后台线程第5次执行！ 
后台线程第6次执行！ 
后台线程第7次执行！ 


 
从上面的执行结果可以看出：
前台线程是保证执行完毕的，后台线程还没有执行完毕就退出了。

关于守护线程的实例：[http://jiangnan31.github.io/2014/10/28/Java-多线程 <一>/](http://jiangnan31.github.io/2014/10/28/Java-%E5%A4%9A%E7%BA%BF%E7%A8%8B-%E4%B8%80/)    

实际上：JRE判断程序是否执行结束的标准是所有的前台执线程行完毕了，而不管后台线程的状态，因此，在使用后台线程时候一定要注意这个问题。