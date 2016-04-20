title: Future、FutureTask、Callable
date: 2014-10-28 14:13:50
categories: 笔记
tags: Concurrency
---
## 1. Future接口  
JDK API中的说明
```java
public interface Future <V>
```
 Future表示异步计算的结果。<span style="color:red">它提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。</span>计算完成后只能使用 get 方法来获取结果，如有必要，计算完成前可以阻塞此方法。<span style="color:red">取消则由cancel方法来执行。</span>还提供了其他方法，以确定任务是正常完成还是被取消了。一旦计算完成，就不能再取消计算。如果为了可取消性而使用 Future 但又不提供可用的结果，则可以声明 Future<?> 形式类型、并返回 null 作为底层任务的结果。 
<!--more-->
方法：
（1）
```java
boolean cancel(boolean mayInterruptIfRunning)
```
试图取消对此任务的执行。如果任务已完成、或已取消，或者由于某些其他原因而无法取消，则此尝试将失败。当调用 cancel 时，如果调用成功，而此任务尚未启动，则此任务将永不运行。如果任务已经启动，则 mayInterruptIfRunning 参数确定<span style="color:red">是否应该以试图停止任务的方式来中断执行此任务的线程。</span> 此方法返回后，对 isDone() 的后续调用将始终返回 true。如果此方法返回 true，则对 isCancelled() 的后续调用将始终返回 true。 
(2)
```java
boolean isCancelled()
```
如果在任务正常完成前将其取消，则返回 true。 
(3)
```java
boolean isDone()
```
如果任务已完成，则返回 true。 可能由于正常终止、异常或取消而完成，在所有这些情况中，此方法都将返回 true。 
(4)
```java
V get() throws InterruptedException,ExecutionException
```
如有必要，等待计算完成，然后<span style="color:red">获取其结果。</span> 
(5)
```java
V get(long timeout,TimeUnit unit) throws InterruptedException,ExecutionException,TimeoutException
```
如有必要，最多等待为使计算完成所给定的时间之后，获取其结果（如果结果可用）。 
## 2. RunnableFuture接口

```java
public interface RunnableFuture<V> extends Runnable, Future<V>
```
作为 Runnable 的 Future。成功执行 run 方法可以完成 Future 并允许访问其结果。

继承了Runnable方法和Future方法，所以实现了这个接口的类可以被多线程下执行，并且能够异步的取得结果。
## 3. Callable接口
callable接口与Runnable接口非常相似，他们都是为潜在的可能被多线程调用的类所设计。但是callable的call（）方法是有返回值的，而且可以抛出一个Check Exception异常。
## 4. Runnable接口
Runnable接口：这个接口是我们非常熟悉的，在这个接口中只有一个run（）方法，在这个方法中我们可以实现我们的计算逻辑。但是没有返回值，也就是说runnable接口只管运行，不管运行结果。
## 5. FutureTask

```java
public class FutureTask<V> extends Object implements RunnableFuture<V>
```
FutureTask类是Future的一个实现，并实现了Runnable，所以可通过 Excutor和Thread对象执行。

下面的例子可以做为参考：
```java
import java.util.concurrent.Callable;

public class Changgong implements Callable<Integer>{

    private int hours=12;
    private int amount;
    
    public Integer call() throws Exception {
        while(hours>0){
            System.out.println("I'm working......  "+hours+" hours");
            amount ++;
            hours--;
            Thread.sleep(1000);
        }
        return amount;
    }
}
```

```java
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
public class Dizhu {
        
    public static void main(String args[]){
        Changgong worker = new Changgong();
        FutureTask<Integer> jiangong = new FutureTask<Integer>(worker);
        new Thread(jiangong).start();
        while(!jiangong.isDone()){
            try {
                System.out.println("看长工做完了没...");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        int amount;
        try {
            amount = jiangong.get();
            System.out.println("工作做完了,上交了"+amount);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (ExecutionException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        
        
    }
}
```
运行结果：
看长工做完了没...
I'm working......  12 hours
看长工做完了没...
I'm working......  11 hours
看长工做完了没...
I'm working......  10 hours
看长工做完了没...
I'm working......  9 hours
看长工做完了没...
I'm working......  8 hours
看长工做完了没...
I'm working......  7 hours
看长工做完了没...
I'm working......  6 hours
看长工做完了没...
I'm working......  5 hours
看长工做完了没...
I'm working......  4 hours
看长工做完了没...
I'm working......  3 hours
看长工做完了没...
I'm working......  2 hours
看长工做完了没...
I'm working......  1 hours
看长工做完了没...
工作做完了,上交了12
