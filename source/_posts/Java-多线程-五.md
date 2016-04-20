title: "Java 多线程<五> 线程的交互&&线程的调度-休眠(转载)"
date: 2014-12-09 15:44:19
categories: 转载
tags: Concurrency
---
# 线程的交互

## 一、线程交互的基础知识
 
线程交互知识点需要从java.lang.Object的类的三个方法来学习：
 
(1) void notify()    
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;唤醒在此对象监视器上等待的单个线程。 
(2) void notifyAll()   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;唤醒在此对象监视器上等待的所有线程。 
(3) void wait() 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。

<!--more-->

当然，wait()还有另外两个重载方法：

(1) void wait(long timeout) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。 
(2) void wait(long timeout, int nanos) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。
 
以上这些方法是帮助线程传递线程关心的时间状态。
 
关于等待/通知，要记住的关键点是：
必须从同步环境内调用wait()、notify()、notifyAll()方法。<span style="color:red">线程不能调用对象上等待或通知的方法，除非它拥有那个对象的锁。</span>
wait()、notify()、notifyAll()都是Object的实例方法。与每个对象具有锁一样，每个对象可以有一个线程列表，他们等待来自该信号（通知）。线程通过执行对象上的wait()方法获得这个等待列表。从那时候起，它不再执行任何其他指令，直到调用对象的notify()方法为止。如果多个线程在同一个对象上等待，则将只选择一个线程（不保证以何种顺序）继续执行。如果没有线程等待，则不采取任何特殊操作。
 
下面看个例子就明白了：
```java
/** 
* 计算1+2+3 ... +100的和 
* 
* @author leizhimin 2008-9-15 13:20:49 
*/ 
class ThreadB extends Thread { 
    int total; 

    public void run() { 
        synchronized (this) { 
            for (int i = 0; i < 101; i++) { 
                total += i; 
            } 
            try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
            //（完成计算了）唤醒在此对象监视器上等待的单个线程，在本例中线程A被唤醒 
            notify(); 
        } 
    } 
}
/** 
* 计算输出其他线程锁计算的数据 
* 
* @author leizhimin 2008-9-15 13:20:38 
*/ 
public class ThreadA { 
    public static void main(String[] args) { 
        ThreadB b = new ThreadB(); 
        //启动计算线程 
        b.start(); 
        //线程A拥有b对象上的锁。线程为了调用wait()或notify()方法，该线程必须是那个对象锁的拥有者 
        synchronized (b) { 
            try { 
                System.out.println("等待对象b完成计算。。。"); 
                //当前线程A等待 
                b.wait(); 
            } catch (InterruptedException e) { 
                e.printStackTrace(); 
            } 
            System.out.println("b对象计算的总和是：" + b.total); 
        } 
    } 
}
```
**Result:**
等待对象b完成计算。。。 
b对象计算的总和是：5050 

Process finished with exit code 0

个人理解，不知道对错。。。
>对象b自身具有锁，同时也存有一个线程列表，他们等待来自该信号（通知）。上述代码中的主线程通过执行对象b上的wait()方法获得这个等待列表。从那时候起，它不再执行任何其他指令，直到调用对象的notify()方法为止。
 
千万注意：
当在对象上调用wait()方法时，执行该代码的线程立即放弃它在对象上的锁。然而<span style="color:red">调用notify()时，并不意味着这时线程会放弃其锁。</span>如果线程仍然在完成同步代码，则线程在移出之前不会放弃锁。因此，只要调用notify()并不意味着这时该锁变得可用。
 
二、多个线程在等待一个对象锁时候使用notifyAll()
 
在多数情况下，最好通知等待某个对象的所有线程。如果这样做，可以在对象上使用notifyAll()让所有在此对象上等待的线程冲出等待区，返回到可运行状态。
 
下面给个例子：
```java

class Calculator extends Thread { 
        int total; 

        public void run() { 
                synchronized (this) { 
                        for (int i = 0; i < 101; i++) { 
                                total += i; 
                        } 
                } 
                //通知所有在此对象上等待的线程 
                notifyAll(); 
        } 
}

public class ReaderResult extends Thread { 
        Calculator c; 

        public ReaderResult(Calculator c) { 
                this.c = c; 
        } 

        public void run() { 
                synchronized (c) { 
                        try { 
                                System.out.println(Thread.currentThread() + "等待计算结果。。。"); 
                                c.wait(); 
                        } catch (InterruptedException e) { 
                                e.printStackTrace(); 
                        } 
                        System.out.println(Thread.currentThread() + "计算结果为：" + c.total); 
                } 
        } 

        public static void main(String[] args) { 
                Calculator calculator = new Calculator(); 

                //启动三个线程，分别获取计算结果 
                new ReaderResult(calculator).start(); 
                new ReaderResult(calculator).start(); 
                new ReaderResult(calculator).start(); 
                //启动计算线程 
                calculator.start(); 
        } 
}
``` 
运行结果：
Thread[Thread-1,5,main]等待计算结果。。。 
Thread[Thread-2,5,main]等待计算结果。。。 
Thread[Thread-3,5,main]等待计算结果。。。 
Exception in thread "Thread-0" java.lang.IllegalMonitorStateException: current thread not owner 
  at java.lang.Object.notifyAll(Native Method) 
  at threadtest.Calculator.run(Calculator.java:18) 
Thread[Thread-1,5,main]计算结果为：5050 
Thread[Thread-2,5,main]计算结果为：5050 
Thread[Thread-3,5,main]计算结果为：5050 

Process finished with exit code 0 
 
运行结果表明，程序中有异常，并且多次运行结果可能有多种输出结果。这就是说明，这个多线程的交互程序还存在问题。究竟是出了什么问题，需要深入的分析和思考，下面将做具体分析。

{
根据jdk的void notifyAll()的描述，“解除那些在该对象上调用
wait()方法的线程的阻塞状态。
该方法只能在同步方法或同步块内部调用。
如果当前线程不是对象所得持有者，该方法抛出一个
java.lang.IllegalMonitorStateException 异常”
}
 
实际上，上面这个代码中，我们期望的是读取结果的线程在计算线程调用notifyAll()之前等待即可。 但是，如果计算线程先执行，并在读取结果线程等待之前调用了notify()方法，那么又会发生什么呢？这种情况是可能发生的。因为无法保证线程的不同部分将按照什么顺序来执行。幸运的是当读取线程运行时，它只能马上进入等待状态----它没有做任何事情来检查等待的事件是否已经发生。  ----因此，如果计算线程已经调用了notifyAll()方法，那么它就不会再次调用notifyAll()，----并且等待的读取线程将永远保持等待。这当然是开发者所不愿意看到的问题。
 
因此，当等待的事件发生时，需要能够检查notifyAll()通知事件是否已经发生。通常，解决上面问题的最佳方式是利用某种循环，该循环检查某个条件表达式，只有当正在等待的事情还没有发生的情况下，它才继续等待。


# 线程的调度-休眠

Java线程调度是Java多线程的核心，只有良好的调度，才能充分发挥系统的性能，提高程序的执行效率。
 
这里要明确的一点，不管程序员怎么编写调度，<span style= "color:red">只能最大限度的影响线程执行的次序，而不能做到精准控制。</span>
 
线程休眠的目的是使线程让出CPU的最简单的做法之一，线程休眠时候，会将CPU资源交给其他线程，以便能轮换执行，当休眠一定时间后，线程会苏醒，进入准备状态等待执行。
 
线程休眠的方法是Thread.sleep(long millis) 和Thread.sleep(long millis, int nanos) ，均为静态方法，那调用sleep休眠的哪个线程呢？简单说，哪个线程调用sleep，就休眠哪个线程。
```Java
/** 
* Java线程：线程的调度-休眠 
* 
* @author leizhimin 2009-11-4 9:02:40 
*/ 
public class Test { 
        public static void main(String[] args) { 
                Thread t1 = new MyThread1(); 
                Thread t2 = new Thread(new MyRunnable()); 
                t1.start(); 
                t2.start(); 
        } 
} 

class MyThread1 extends Thread { 
        public void run() { 
                for (int i = 0; i < 3; i++) { 
                        System.out.println("线程1第" + i + "次执行！"); 
                        try { 
                                Thread.sleep(50); 
                        } catch (InterruptedException e) { 
                                e.printStackTrace(); 
                        } 
                } 
        } 
} 

class MyRunnable implements Runnable { 
        public void run() { 
                for (int i = 0; i < 3; i++) { 
                        System.out.println("线程2第" + i + "次执行！"); 
                        try { 
                                Thread.sleep(50); 
                        } catch (InterruptedException e) { 
                                e.printStackTrace(); 
                        } 
                } 
        } 
}
```
线程2第0次执行！ 
线程1第0次执行！ 
线程1第1次执行！ 
线程2第1次执行！ 
线程1第2次执行！ 
线程2第2次执行！ 


 
从上面的结果输出可以看出，无法精准保证线程执行次序。
转载自：[http://lavasoft.blog.51cto.com/62575/27069/](http://lavasoft.blog.51cto.com/62575/27069/)