---
title: Java多线程面试汇总
date: 2022-02-24 18:35:13
updated: 2022-02-24 18:35:13
tags: 面试整合（二）
categories: Java
---

# 三、多线程

## 1. 并行和并发的区别

+ 并行：多个处理器或多核处理器同时处理多个任务；
+ 并发：多个任务在一个CPU按细分的时间片轮流（交替）执行，从逻辑上来看是同时执行；



## 2. 线程与进程的区别

一个程序至少有一个进程，一个进程至少有一个线程，一个进程下可以有多个线程来增加执行程序的速度；



## 3. 守护线程

运行在后台的一种特殊进程，独立于控制终端并且周期性执行某种任务或等待处理某些发生的事情。例如java中垃圾回收的线程就是特殊的守护线程；



## 4. 创建线程的方式

+ 继承Thread类重写run方法；
+ 实现Runnable接口也是执行run方法；
+ 实现Callable接口；

已继承别的类的情况下想要实现多线程，只能使用接口实现；Runnable没有返回值，Callable可以拿到返回值。因此Callable可以看做是Runnable的补充；



## 5. 线程有哪些状态

+ NEW 尚未启动
+ RUNNABLE 正在执行中
+ BLOCKED 阻塞（被同步锁或者IO锁阻塞）
+ WAITING 永久等待状态
+ TIME_WAITED 等待指定时间重新被唤醒
+ TERMINATED 执行完成



## 6. 线程的生命周期

![image-20220224190305461](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220224190305461.png)



## 7. sleep()和wait()的区别

+ 类不同：sleep()来自Thread，wait()来自Object
+ 释放锁：sleep()不释放锁，wait()释放锁；
+ 用法不同：sleep()时间到会自动恢复，wait()可以使用notify() / notifyAll()直接唤醒



## 8. notify() / notifyAll()的区别

+ notifyAll()会唤醒所有线程，notify()之后只会唤醒一个线程；
+ notifyAll()调用后，会将全部线程由等待转移到锁池，然后参与锁竞争，竞争成功后继续执行；不成功则留在锁池等待锁释放后再次参与竞争；
+ notify()只会唤醒一个线程，具体唤醒哪一个有虚拟机控制；



## 9. run()和start()的区别

start()适用于启动线程，run()方法用于执行线程的运行时代码，run()可以重复调用，start()只调用一次；



## 10. 创建线程池的方式

+ newSingleThreadExecutor()：它的特点在于工作线程数目被限制为 1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目；
+ newCachedThreadPool()：它是一种用来处理大量短时间工作任务的线程池，具有几个鲜明特点：它会试图缓存线程并重用，当无缓存线程可用时，就会创建新的工作线程；如果线程闲置的时间超过 60 秒，则被终止并移出缓存；长时间闲置时，这种线程池，不会消耗什么资源。其内部使用 SynchronousQueue 作为工作队列；
+ newFixedThreadPool(int nThreads)：重用指定数目（nThreads）的线程，其背后使用的是无界的工作队列，任何时候最多有 nThreads 个工作线程是活动的。这意味着，如果任务数量超过了活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数目 nThreads；
+ newSingleThreadScheduledExecutor()：创建单线程池，返回 ScheduledExecutorService，可以进行定时或周期性的工作调度；
+ newScheduledThreadPool(int corePoolSize)：和newSingleThreadScheduledExecutor()类似，创建的是个 ScheduledExecutorService，可以进行定时或周期性的工作调度，区别在于单一工作线程还是多个工作线程；
+ newWorkStealingPool(int parallelism)：这是一个经常被人忽略的线程池，Java 8 才加入这个创建方法，其内部会构建ForkJoinPool，利用Work-Stealing算法，并行地处理任务，不保证处理顺序；
+ ThreadPoolExecutor()：是最原始的线程池创建，上面1-3创建方式都是对ThreadPoolExecutor的封装。

```java
// 以其中一个为例子，后面相同
ExecutorService threadPool = Executors.newFixedThreadPool(5);
```



## 11. 线程池的状态

+ RUNNING：这是最正常的状态，接受新的任务，处理等待队列中的任务。
+ SHUTDOWN：不接受新的任务提交，但是会继续处理等待队列中的任务。
+ STOP：不接受新的任务提交，不再处理等待队列中的任务，中断正在执行任务的线程。
+ TIDYING：所有的任务都销毁了，workCount 为 0，线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()。
+ TERMINATED：terminated()方法结束后，线程池的状态就会变成这个。



## 12. 线程池中submit()和execute()的方法区别

+ execute()：只能执行 Runnable 类型的任务。
+ submit()：可以执行 Runnable 和 Callable 类型的任务。

Callable类型的任务可以获取执行的返回结果，而Runnable执行无返回值；



## 13. Java如何保证多线程运行安全

+ 使用安全类，比如java.util.concurrent下的类；
+ 使用自动锁synchronized；
+ 使用手动锁lock;

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    System.out.println("获得锁");
} catch (Exception e) {
    //todo: handle exception
} finally {
    System.out.println("释放锁");
    lock.unlock();
}
```



## 14. 多线程中synchronized锁升级原理是什么

在锁对象的对象头里面有一个 threadid 字段，在第一次访问的时候 threadid 为空，jvm 让其持有偏向锁，并将 threadid 设置为其线程 id，再次进入的时候会先判断 threadid 是否与其线程 id 一致，如果一致则可以直接使用此对象，如果不一致，则升级偏向锁为轻量级锁，通过自旋循环一定次数来获取锁，执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁，此过程就构成了 synchronized 锁的升级。

锁升级是为了减低了锁带来的性能消耗。在 Java 6 之后优化 synchronized 的实现方式，使用了偏向锁升级为轻量级锁再升级到重量级锁的方式，从而减低了锁带来的性能消耗。



## 15. 死锁

### 什么是死锁

当线程A持有独占锁a，并尝试去获取独占锁b的同时，线程B持有独占锁b，并尝试获取独占锁a的情况下，就会发生AB两个线程由于互相持有对方需要的锁，而发生的阻塞现象，我们称为死锁。

### 如何防范死锁

+ 尽量使用 tryLock(long timeout, TimeUnit unit)的方法(ReentrantLock、ReentrantReadWriteLock)，设置超时时间，超时可以退出防止死锁。
+ 尽量使用 java.util.concurrent 并发类代替自己手写锁。
+ 尽量降低锁的使用粒度，尽量不要几个功能用同一把锁。
+ 尽量减少同步的代码块。

#### 

## 16. ThreadLocal是什么，有哪些应用场景

为每个使用该变量的线程提供独立的变量副本，每个线程都可以独立改变自己的副本，并且不会影响其他线程对应的副本；



## 17. 说一下atomic原理

利用CAS（Compare And Swap）、volatile和native方法保证原子操作，从而避免synchronized的高开销，执行效率大大提升；



# 18. 讲讲进程、线程和协程的区别

## 进程

进程是具有一定独立功能的程序在一个数据集上的一次动态执行过程，是操作系统分配和调度的独立单位。也是应用程序的载体。

进程一般由程序、数据集和进程控制块三部分组成。

+ 程序用于描述进程需要完成的功能；
+ 数据集是程序在执行过程中所需要的数据和工作区；
+ 程序控制块包括进程的描述信息和控制信息，是进程存在的唯一标识；

进程具有下列特征：

+ 动态性：进程是程序的一次执行过程，是临时的、有生命周期的、动态产生/消亡的；
+ 并发性：任何进程都可以同其他进程一起并发执行；
+ 独立性：进程是系统进行数据分配和调度的一个独立单位；
+ 结构性：进程由程序、数据和进程控制块三部分组成；



## 线程

线程是程序执行中的一个单一顺序控制流程，是程序执行流的最小单元，是处理器调度和分派的基本单位。一个进程可以拥有一个或多个线程，各个线程之间共享程序的内存空间（也就是所在进程的内存空间）。



## 进程和线程的区别

+ 线程是程序执行的最小单位，进程是操作系统分配资源的最小单位；
+ 一个进程由一个或多个线程组成，线程是一个进程代码的不同执行路线；
+ 进程之间相互独立，同一个进程下的各个线程之间共享程序的内存空间；
+ 调度和切换：线程的上下文切换要比进程的上下文切换快得多；

![image-20220302152529598](https://humphrey-blogs-bucket.oss-cn-shenzhen.aliyuncs.com/img/image-20220302152529598.png)



## 协程

一种基于线程之上但又比线程更加轻量级的存在，这种由程序自己写程序来管理的轻量级线程叫做“用户空间线程”
