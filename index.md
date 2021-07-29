# **JUC笔记**
[JUC  volatile关键字](https://github.com/wuhefei/wuhefei.github.io/blob/main/JUC/volatile.md)

[Spring-data-jpa 查询  复杂查询陆续完善中](http://www.cnblogs.com/sxdcgaq8080/p/7894828.html)

[Spring-data-jpa 查询  复杂查询陆续完善中](http://www.cnblogs.com/sxdcgaq8080/p/7894828.html)

# **上下文切换**

上下文切换是非常耗效率的。

采用无锁编程，比如将数据按照 Hash(id) 进行取模分段，每个线程处理各自分段的数据，从而避免使用锁。

采用 CAS(compare and swap) 算法，如 Atomic 包就是采用 CAS 算法。

合理的创建线程，避免创建了一些线程但其中大部分都是处于 waiting 状态，因为每当从 waiting 状态切换到 running 状态都是一次上下文切换。

# **死锁**

死锁的场景一般是：线程 A 和线程 B 都在互相等待对方释放锁，或者是其中某个线程在释放锁的时候出现异常如死循环之类的。这时就会导致系统不re可用。

常用的解决方案如下：

尽量一个线程只获取一个锁。

一个线程只占用一个资源。

尝试使用定时锁，至少能保证锁最终会被释放。

  2.1 专门的算法，原则

  2.2 尽量避免减少同步资源的定义

  2.3 尽量避免嵌套同步。

# **资源限制**

当在带宽有限的情况下一个线程下载某个资源需要 1M/S,当开 10 个线程时速度并不会乘 10 倍，反而还会增加时间，毕竟上下文切换比较耗时。

如果是受限于资源的话可以采用集群来处理任务，不同的机器来处理不同的数据，就类似于开始提到的无锁编程。

Java 内存模型

volatile

1. 保持可见性
2. 保持有序性
3. 不能保证原子性

Sychronized 互斥锁

锁对象： 非静态 this ，静态Class 括号Object

预防死锁

互斥：不能破坏

占有且等待：同时申请所有资源

不可抢占：sychronized 解决不了 lock可以解决

循环等待：给资源设置id 字段，每次都是按顺寻申请锁的

等待通知机制

wait notify notifyAll

线程的性能指标

延迟

吞吐量

最佳线程数

CPU密集型 ：线程数=CPU+1

IO密集型 ：（IO耗时/CPU耗时+1）\*CPU核数

![](RackMultipart20210728-4-1tzx3pv_html_138208abfe3575c9.png)

JDK 并发包

**Lock**** ： ****Lock unlock**

互斥锁,和sychrnized 一样的功能，里面能保证可见性

**Condition**** ： **** await ****，**** signal**

条件：相比于sychronized的Object.wait Condition可以实现多条件唤醒机制。

**Semaphore**  **：**** acquire **** ， ****release**

信号量：可以用来实现多个线程访问一个临界区,如实现对象池设计中的限流器

**ReadWriteLock**  **：**** readLock writeLock**

写锁，读锁：允许多线程读，一个线程写，写锁持有时所有读锁和写锁都阻塞（写锁的获取要等所有读写锁释放）

适用于读多写少的场景

**StampedLock**** ： ****tryOptimisticRead validate**

写锁，读锁 （分悲观读锁，乐观读锁）

**线程同步**

CountDownLatch 一个线程等待多个线程

1. 初始化 --\&gt; countDown（减1） --\&gt; await（等待为0）

CycliBarrier：一组线程之间的相互等待

1. 初始化 --\&gt; 设置回调函数（为0时执行，并返回原始值） --\&gt;  await（减1并等待 为0）

**并发容器**

List：

**CopyOnWriteArrayList**** ：适用于写少的场景，要容忍可能的读不一致**

**Map**** ：**

**ConcurrentHashMap:**** 分段锁**

**ConcurrentSkipListMap**  **：跳表**

**Set**** ：**

**CopyOnWirteArryset**** ：同上**

**CopyCurrentSkipListSet**** ：同上**

**Queue**** ：**

**分类：阻塞**** Blocking **** 单端 ****Queue**  **双端**** Queue**

**单端阻塞（**** BlockingQueue ****）** ：Array～、Linked～、Sychronized～、LinkedTransfer～、Priority～、Delay～

**双端阻塞（**** BlockingDeque ****）** ：Linked～

**单端非阻塞（**** Queue ****）** ：ConcurrentLinked～

**双端非阻塞（**** Deque ****）** ：ConcurrentLinked～

**原子类**

无锁方案原理：增加了硬件支持。即CPU的CAS指令

AbA问题：有解决AbA问题需求的时候，增加一个递增的版本号纬度化解。

循环时间开销大：自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

只能保证一个共享变量的原子操作：把多个变量放到一个变量中

分类：原子化基本数据类型，原子化引用类型，原子化数组，原子化对象属性更新器，原子化累加器

Future：cancel ，isCanceled isDone，get

FutureTask ： 实现了呢哇runnable接口和Future接口

**强大的工具类**

CompletableFuture :一个强大的异步编程工具类（任务之间有聚合功能）

CompletionService：批量并行任务

在不同的进程当中

分布式锁

基于数据库的锁

可以创建一张表，其中的某个字段设置成为唯一索引，当多个请求过来的时候之哦于心间记录成功的请求才算获取到锁，当使用完毕删除这条记录的时候即可释放锁

存在的问题：

数据单点问题，挂了怎么办？

不是重入锁，同一进程中无法释放锁之前再次添加锁，因为数据库中也已经存了一条记录了

锁是非阻塞的 一旦insert 失败则会立即返回，并不会进入阻塞队列只能下一次再次获取

锁没有失效时间，如果那个进程解锁失败那就没有请求可以再次获取锁了

解决方案：

数据库切换主从，不存在单点情况

在表中添加一个同步状态字段，每次获取锁时 +1 释放锁的时 -1 当状态为0的时候就删除这条记录，即释放锁

非阻塞的情况可以用while循环来实现，循环的时候记录时间，达到X秒记为超时，break

可以开启一个定时任务每隔一段时间扫描找出多少X秒都没有被删除的记录，主动删除这条记录

基于redis

使setNX(key) setEX(timeout)命令，只有在该key不存在的时候创建整个key ，就相当于获取了锁，由于有超时时间，所以过了规定时间会自动删除，这样也可以避免死锁。

基于zookeeper

# **线程池**

设计原理

用生产者消费者模型，线程池是消费者，调用这是生产者

线程池对象里维护一个阻塞队列，一个已经跑起来的工作线程组 ThreadsList

```
package com.sirchw;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class MyThreadPool {
    BlockingQueue<Runnable> workQueue;
    List<workerThread> threads = new ArrayList<>();

    MyThreadPool(int poolsize,BlockingQueue<Runnable> workQueue){
        this.workQueue=workQueue;
        for (int idx=0;idx<poolsize;idx++){
            workerThread work = new workerThread();
            work.start();
            threads.add(work);
        }
    }
    void execute(Runnable command) throws InterruptedException {
        workQueue.put(command);
    }
    class workerThread extends Thread{
        public void run(){
            while(true){
                Runnable task= null;
                try {
                    task = workQueue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                task.run();
            }
        }
    }
}
```

线程资源必须通过线程池提供，不允许在应用中自行显式创建线程

避免 可能造成的系统创建大量同类线程而导致消耗完内存或者过度切换的问题。

```
Executors.newCachedThreadPool():无限线程池
Executors.newFixedThreadPool() :创建固定大小的线程池
Executors.newSingleThreadExecutor()：创建单个线程的线程池



public static ExecutorService newCachedThreadPool(){
    return new ThreadPoolExecutor(0,Integer.MAX_VALUE,60L,TimeUnit.SECONDS,new SynchronousQueue<Runnable>());
}


//实际上还是利用ThreadPoolExecutor实现的
//先来看看创建线程  api
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) 
//corePoolSize 线程池的基本大小
//maximumPoolSize 线程池最大线程大小
//keepAliveTime 和 unit 则是线程空闲后的存活时间
//workQueue 用于存放任务的阻塞队列。
//handler 当队列和最大线程池都满了之后的饱和策略。
```
