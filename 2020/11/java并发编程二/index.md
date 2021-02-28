# Java并发编程二


# Java并发编程二

## Java并发编程的艺术

### 上下文切换

CPU通过给每个线程分配CPU时间片来实现在单核CPU中执行多线程。时间片是CPU分配给各个线程的时间，因为时间片非常短，所以CPU通过不停切换线程执行，让我们感觉多个线程同时执行（并发）。

CPU通过时间片分配算法来循环执行任务，当一个任务的时间片用完，会保留当前任务的所处状态，执行下一个任务的时间片，而执行完下一个时间片后会切回这个任务，在加载这个任务的状态。所以任务从保存到再加载就是一次**上下文的切换。**

#### 多线程就一定快吗？

不一定，因为线程的创建有上下文的切换。

#### 如何减少上下文切换？

* 无锁并发编程。多线程竞争锁的时候，会引起上下文的切换，所以多线程编程的时候可以尝试无锁并发，让每个线程处理不同的数据段数据。
* CAS算法。Java的Atomic包使用CAS算法来更新数据，而不需要锁。

> CAS（Compare And Swap）一种无锁算法。
>
> 涉及三个操作数：预期值、读写的内存位置、需要写入的新值
>
> 会产生三个问题：ABA问题（变量前添加版本号），循环时间开销大（需要不断自旋来读取最新的内存值），只能保证一个共享变量的原子操作。

* 使用最少的线程，避免使用无效线程。
* 协程：在单线程离实现多任务调度，并在单线程离维持多个任务的切换

## Java内存模型

## 前言

**并发编程模型的两个关键问题：**

1. 线程之间如何通信
2. 线程之间如何同步

通信指的是线程之间以何种机制来交换信息。在命令式编程过程中，线程之间的通信机制有两种：

* 共享内存
* 消息传递

同步指程序中用于控制不同线程间操作发生的相对顺序的机制。

Java的并发采用的事共享内存模型，Java线程之间的通信总是隐式进行的 。

## Java内存模型的抽象概念

在Java中，所有实例域、静态域、数组元素等都存储在堆内存中，堆内存在线程之间共享（共享变量）

线程之间通信的两个步骤：

1. 线程A把本地内存A中更新过的共享变量刷新到共享内存中去
2. 线程B到主内存中读取线程A更新过的共享变量

### 指令序列的重排列

重排列分三种类型，同时也是Java源代码会经历的：

1. 编译器优化的重排列（会出现内存可见性问题）
2. 指令级并行的重排列
3. 内存系统的重排列

JMM属于语言级别的内存模型，确保在不同的编译器和不同的处理器平台上，通过禁止特定类型的编译器重排列和处理器重排列，提供一直的内存可见性保证。

### happens-before

是JMM的核心。

* 会改变程序执行结果的重排列：禁止
* 不会改变程序执行结果的重排列：不做要求

JMM对编译器和处理器的要求：即不改变程序执行结果，编译器和处理器如何优化都可以。

#### happens-before规则

1. 程序顺序规则

2. 监视器规则

3. volatile规则：单例双重锁检测问题volatile的作用

4. 传递性

5. start（）规则：如果线程A执行ThreadB.start()启动，那么线程A的TreadB.start()操作 happens-before 于线程B中任意操作

6. join（）规则：如果线程A执行ThreadB.join()并成功返回，那么线程B的任意操作happens-before于线程A从TreadB.join()操作成功返回


#### 双重检查锁定与延迟初始化

双重检查锁

```java
public class SingletonLanHan03 {

    private static volatile SingletonLanHan03 instance;

    private SingletonLanHan03() {}

    public static SingletonLanHan03 getInstance() {
        // 判断对象是否实例化过，没有才进入，如果已经实例化过，则无需进行初始化，减少开销
        if(instance == null) {
            // 没有实例化，对其类对象进行加锁
            synchronized (SingletonLanHan03.class) {
                if (instance == null) {
                    // 进行实例化，但注意这里是非原子性操作
                    instance = new SingletonLanHan03();
                }
            }
        }
        return instance;
    }
}

```

将Instance使用volatile修饰的原因：

当线程执行到第一次检查的时候，instance不为null，只是可能未完成初始化操作。

创建一个对象的三个步骤：

1. 为该对象分配空间
2. 初始化该对象
3. 设置该对象的引用指针指向内存分配的地址

但JVM可能会进行指令重排序（JIT）从而更改了2、3的顺序，可能该对象已经创建了，但还未来的及进行初始化，便又有线程进行实例的调用了。而volatile防止了指令重排列。

#### 基于类初始化的解决方案

```java
public class SingletonInClass {
    private SingletonInClass() {}

    private static class SingletonHolder {
        private static SingletonInClass instance = new SingletonInClass();
    }

    public static SingletonInClass getInstance() {
        return SingletonHolder.instance;
    }
}

```

JVM在类的初始化阶段（即Class被加载后，且被线程使用之前）会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁会同步多个线程对该类的初始化过程。

即使发生在指令重排序，但其他线程看不到。

### 线程状态

* new 初始化
* runnable 运行状态
* blocked 阻塞状态
* waiting 等待状态
* time_waiting 超时等待 （不同于waiting，在指定时间后自动返回sleep（））
* terminated 终止

#### notify（）、notifyAll（）、wait（）

1. 使用这三个方法都需要先对调用对象加锁
2. 调用wait（）方法后，线程状态从runnable转换为waiting，并将当前对象放入到该对象的等待池中
3. notify（）和notifyAll（）后，等待线程会需要等待调用方法的线程释放当前对象后与其他对象竞争
4. notify（）会随机将等待池（等待队列）中的线程移动一个到锁池（同步队列）中去，而notifyAll（）会将等待池中全部线程移动到锁池去，被移动的线程状态从waiting->blocked
5. 从wait（）方法返回的前提是获取到了该对象的锁

#### Thread.join（）的使用

当线程A等待thread线程终止之后才从thread.join()返回。

### ThreadLocal的使用

ThreadLocal，即线程变量，是一个以ThreadLocal对象为键、任意对象为值的结构。即一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值。

```java
package Advance.concurrent_programming.thread_create;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/10/20
 **/
public class ThreadLocalTest {
    static ThreadLocal<String> localVariable = new ThreadLocal<String>();

    static void print(String str) {
        // 打印当前localVariable变量的值
        System.out.println(str + ":" + localVariable.get());
        // 清除当前本地内存中localVariable变量
        // localVariable.remove();
    }

    public static void main(String[] args) {
        // 创建线程one
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                // 设置线程one中本地变量localVariable的值
                localVariable.set("threadOne local variable");
                // 调用打印函数
                print("threadOne");
                // 打印本地变量值
                System.out.println("threadOne remove after" + ":" + localVariable.get());
            }
        });
        // 创建线程two
        Thread threadTwo = new Thread(new Runnable() {
            @Override
            public void run() {
                localVariable.set("threadTwo local variable");
                print("threadTwo");
                System.out.println("threadTwo remove after" + ":" + localVariable.get());
            }
        });
        threadOne.start();
        threadTwo.start();
    }
}

```

### 线程池

线程池解决了大量从客户端传入执行时间短，需要服务端快速处理并返回结果，线程上下文切换带来的开销。

线程池先预先创建若干数量的线程，并且不能由用户直接对线程的创建进行控制，在这个前提下重复使用固定或较为固定数目的线程来完成任务的执行。

Java中的线程池类就是一个种生产者和消费者模式的实现方式，生产者把任务丢给线程池，线程池创建线程并处理任务（消费）

```java
package Advance.concurrent_programming.thread_create;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/11/16
 **/
public interface ThreadPool<Job extends Runnable> {
    // 执行一个job
    void execute(Job job);

    // 关闭线程池
    void shutdown();

    // 添加工作线程
    void addWorkers(int num);

    // 关闭工作线程
    void removeWorkers(int num);

    // 获取正在等待执行的线程数量
    int getJobSize();
}

```

#### 线程池的具体实现

```java
package Advance.concurrent_programming.thread_pool;

import sun.tools.jconsole.Worker;

import java.util.ArrayList;
import java.util.Collections;
import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/11/16
 **/
public class DefaultThreadPool<Job extends Runnable> implements ThreadPool<Job> {
    // 线程池的最大限制数目
    private static final int MAX_WORKER_NUMBERS = 10;
    //线程池的默认连接数目
    private static final int DEFAULT_WORKER_NUMBERS = 5;
    // 线程池的最小数量
    private static final int MIN_WORKER_NUMBERS = 1;
    // 工作列表
    private final LinkedList<Job> jobs = new LinkedList<>();
    // 工作者列表
    private final List<Worker> workers = Collections.synchronizedList(new ArrayList<>());
    // 工作者线程数目
    private int workerNum = DEFAULT_WORKER_NUMBERS;
    // 线程编号生成
    private AtomicLong threadNum = new AtomicLong();

    public DefaultThreadPool() {
    }

    public DefaultThreadPool(int workerNum) {
        this.workerNum = workerNum > MAX_WORKER_NUMBERS ? MAX_WORKER_NUMBERS : Math.max(workerNum, MIN_WORKER_NUMBERS);
        initializeWorkers(workerNum);
    }

    // 初始化线程工作
    private void initializeWorkers(int workerNum) {
        for (int i = 0; i < workerNum; i++) {
            Worker worker = new Worker();
            worker.add(worker);
            Thread thread = new Thread(worker, "ThreadPool-Worker-" + threadNum.incrementAndGet());
            thread.start();
        }
    }
    
    @Override
    public void execute(Job job) {
        if (job != null) {
            // 添加一个工作，然后进行通知
            synchronized (job) {
                jobs.addLast(job);
                job.notify();
            }
        }
    }

    @Override
    public void shutdown() {
        for (Worker worker : workers) {
            worker.stopWorker();
        }
    }

    @Override
    public void addWorkers(int num) {
        
    }

    @Override
    public void removeWorkers(int num) {

    }

    @Override
    public int getJobSize() {
        return 0;
    }
}

```

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package sun.tools.jconsole;

import java.util.ArrayList;

public class Worker extends Thread {
    ArrayList<Runnable> jobs = new ArrayList();
    private boolean stopped = false;

    public Worker(String var1) {
        super("Worker-" + var1);
        this.setPriority(4);
    }

    public void run() {
        while(!this.isStopped()) {
            synchronized(this.jobs){}

            while(true) {
                boolean var7 = false;

                Runnable var1;
                try {
                    var7 = true;
                    if (!this.isStopped() && this.jobs.size() == 0) {
                        try {
                            this.jobs.wait();
                        } catch (InterruptedException var8) {
                        }
                        continue;
                    }

                    if (this.isStopped()) {
                        var7 = false;
                        return;
                    }

                    var1 = (Runnable)this.jobs.remove(0);
                    var7 = false;
                } finally {
                    if (var7) {
                        ;
                    }
                }

                var1.run();
                break;
            }
        }

    }

    private synchronized boolean isStopped() {
        return this.stopped;
    }

    public synchronized void stopWorker() {
        this.stopped = true;
        synchronized(this.jobs) {
            this.jobs.notify();
        }
    }

    public void add(Runnable var1) {
        synchronized(this.jobs) {
            this.jobs.add(var1);
            this.jobs.notify();
        }
    }

    public boolean queueFull() {
        synchronized(this.jobs) {
            return this.jobs.size() > 0;
        }
    }
}

```

## Java中的锁

### Lock接口

锁用来控制多个线程访问共享资源的方式。在Lock接口出现之前，Java依靠synchronized来实现锁功能，JDK5之后并发包中新增了Lock接口，提供了与synchronized相似的功能，只是在使用是需要显示地获取和释放锁，但却拥有了锁获取和释放的可操作性，可中断的获取锁以及超时获取锁等多种synchronized关键字不具备的同步特性。

使用方式：

```java
Lock lock = new ReentrantLock();
lock.lock();
try {

}finally {
  lock.unlock();
}
```

synchonized所不具备的特性：

* 尝试非阻塞地获取锁
* 能被中断地获取锁
* 超时获取锁

#### Lock的API

* void lock（）获取锁
* void lockInterruptibly() 可中断地获取锁
* boolean tryLock（）
* boolean tryLock（…）
* void unlock（）释放锁
* Condition newCondition（）等待通知组件

### 重入锁ReentrantLock

就是支持重进入的锁，表示该锁能够支持一个线程对资源的重复加锁，除此之外，该锁的还支持获取锁的公平和非公平性选择。

1. 实现重进入
2. 公平与非公平的区别

> 如果一个锁是公平的那么锁的获取顺序为FIFO

公平锁保证了锁的获取顺序为FIFO，但代价是大量的线程切换，非公平锁可能造成“饥饿”，但极少的线程切换，保证了更大的吞吐量。

#### 读写锁ReentrantReadWriteLock

Mutex和ReentrantLock都是排它锁，这些锁在同一时刻只允许一个线程进行访问。这些锁在同一时刻只允许一个线程进行访问，而读写锁同一时刻允许多个线程进行访问。

##### API

仅有两个方法

* readLock（）
* writeLock（）

```java
        static final int SHARED_SHIFT   = 16;
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
```

读写锁的设计，在一个整形变量上使用按位切割，例如32位的int型，高位的16位为读状态，低位的16位为写状态。

通过S&0000FFFF将高位的16位抹去来获取写状态。得出一个结论：

> S不等于0的时候，当写状态（S&0000FFFF）等于0，则读状态（S>>16)大于0，即读锁已被获取。


