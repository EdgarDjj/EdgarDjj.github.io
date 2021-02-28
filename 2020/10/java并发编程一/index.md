# Java并发编程一


# Java并发编程一

## 并发编程基础

### 线程

进程是代码在数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，线程则是进程的一个执行路径，一个进程至少有一个线程，进程中多个线程共享进程的资源。

### 线程的创建

#### 继承 Thread 类并重写 run 的方法

```java
	// 继承Thread类，并重写run方法
public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("I am a child thread");
        }
    }

    public static void main(String[] args) {
      // 创建线程
        MyThread myThread = new MyThread();
      // 启动线程
        myThread.start();
    }
```

缺点：如果继承了Thread就无法继承其他类，任务和代码没有分离。

#### 实现 Runnable 接口的 run 方法

```java
    public static class RunnableTask implements Runnable {

        @Override
        public void run() {
            System.out.println("I am a child thread");
        }
    }

    public static void main(String[] args) {
//        MyThread myThread = new MyThread();
//        myThread.start();
        RunnableTask task = new RunnableTask();
        new Thread(task).start();
        new Thread(task).start();
    }
```

两个线程公用一个task代码逻辑，如果需要可以给RunableTask添加参数，进行业务区分，且也可以支持继承其他类，但与Thread类重新一样，都没有返回值。

#### 使用 FutureTask 方式

而使用FutureTask的方式可以拿到结果返回值。

```java
public class CallerTask implements Callable<String> {

    @Override
    public String call() throws Exception {
        return "hello world";
    }

    public static void main(String[] args) {
        // 创建异步任务
        FutureTask<String> futureTask = new FutureTask<>(new CallerTask());
        new Thread(futureTask).start();
        String result = null;
        try {
            result = futureTask.get();
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println(result);
    }
}

```

### 线程死锁

#### 什么是线程死锁

死锁是指两个或两个以上的线程在执行过程中，因争夺资源而资源而造成的互相等待的现象，在无外力的作用的情况下，这些线程会一直相互等待而无法继续运行下去。

##### 死锁的条件

* 互斥条件
* **请求并持有资源** 
* 不可剥夺条件
* **环路等待条件**

破坏死锁的条件，只有请求并持有资源和环路等待条件是可以被破坏的

```java
/**
 * Description:
 * 死锁
 * @author:edgarding
 * @date:2020/10/20
 **/
public class DeadLock {
    private static final Object resourceA = new Object();
    private static final Object resourceB = new Object();

    public static void main(String[] args) {
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resourceA) {
                    System.out.println(Thread.currentThread() + " get A ResourceA");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread() + " waiting get ResourceB");
                    synchronized (resourceB) {
                        System.out.println(Thread.currentThread() + "get ResourceB");
                    }
                }
            }
        });

        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (resourceB) {
                    System.out.println(Thread.currentThread() + " get B ResourceB");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread() + " waiting get ResourceA");
                    synchronized (resourceA) {
                        System.out.println(Thread.currentThread() + "get ResourceA");
                    }
                }
            }
        });

        threadA.start();
        threadB.start();
    }
}
输出结果：
Thread[Thread-0,5,main] get A ResourceA
Thread[Thread-1,5,main] get B ResourceB
Thread[Thread-1,5,main] waiting get ResourceA
Thread[Thread-0,5,main] waiting get ResourceB
```

破坏以上死锁条件，仅需要将threadB的持有资源条件改下，和请求的顺序改下。

```java
        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
              // 注意这将resourceB改为resourceA
                synchronized (resourceA) {
                    System.out.println(Thread.currentThread() + " get B ResourceB");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread() + " waiting get ResourceA");
                    synchronized (resourceB) {
                        System.out.println(Thread.currentThread() + "get ResourceA");
                    }
                }
            }
        });
输出结果：
Thread[Thread-0,5,main] get A ResourceA
Thread[Thread-0,5,main] waiting get ResourceB
Thread[Thread-0,5,main]get ResourceB
Thread[Thread-1,5,main] get B ResourceB
Thread[Thread-1,5,main] waiting get ResourceA
Thread[Thread-1,5,main]get ResourceA
```

解决死锁的几个常见方法：

* 避免一个线程同时获得多个锁
* 避免一个线程在锁内同时占有多个资源，尽量保证每个锁只占用一个资源
* 尝试使用定时锁
* 对于数据库锁，加锁去锁都必须在同一个数据库连接里

### 守护线程与用户线程

Java 中的线程分为两类：**daemon 线程**（守护线程）和 **user  线程**（用户线程）

JVM启动时，会调用main函数，main函数所在的线程就是一个用户线程，而在JVM中启动不仅仅只有main这一个用户线程，还会启动许多守护线程，例如：垃圾回收线程。

区别之一就是当最后一个非守护线程结束的时候，JVM会退出，并不管当前是否有守护线程。

调用thread.setDaemon(true) 设置为守护线程。

### ThreadLocal

当创建一个变量后，每个线程访问的时候访问的是自己线程的变量。

ThreadLocal 是JDK 包提供，提供了线程本地变量，也就是如果创建了一个ThreadLocal变量，那么访问这个变量每个线程都会有这个变量的一个本地变量。

#### TreadLocal的使用示例

```java
public class ThreadLocalTest {
    static ThreadLocal<String> localVariable = new ThreadLocal<>();

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
                localVariable.set("thread local variable");
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

#### ThreadLocal实现原理

Thread类中有一个threadLocals和一个inherittableThreadLocals，他们都是ThreadLocalMap类型的变量，而ThreadLocalMap是一个定制化的HashMap，每个线程的变量都是null，只有当前线程第一次调用ThreadLocal的set或者get方法时，才会创建他们。其实每个线程的本地变量不是存放在ThreadLocal实例里面，而是存放在threadLocals变量里面，就是说，ThreadLocal类型的本地变量存放在具体的线程内存空间中。

##### void set（T value）

```java
    public void set(T value) {
      // 获取当前线程
        Thread t = Thread.currentThread();
      // 将当前线程作为key，去查找对应的线程变量，找到则设置
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
          // 第一次调用就创建当前线程对应的HashMap
            createMap(t, value);
    }
```

##### T get（）

```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
      // 如果threadLocals不为null，则返回对应本地变量的值
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

## 并发编程的其它知识

#### 内存可见性问题

当一个线程操作一个共享变量的时候，会将共享变量从主内存中取到自己的工作内存中，处理完后，再将工作变量更新到主内存中。

### synchronized 和 volatile

#### synchronized

synchronized是Java提供的一种原子性内置锁，Java中的每个对象都可以将它当做一个同步锁来进行使用，这些Java内置的使用者看不到的锁，被称为内部锁（监视器锁）。

> 原子性操作Atomcity：指执行一系列操作时，要么全部执行，要么全部不执行，不存在只执行一部分的问题。

线程的代码在进入synchronized的代码块时会自动获取内部锁，这时其它线程范文将会被阻塞挂起。而内置锁是排他锁，当一个线程获得这个锁后，其它线程必须等待该线程释放锁后才能获取该锁。因此会造成一个问题，就是当阻塞一个线程的时候，需要从用户态切换到内核态执行阻塞状态操作，这是很耗时的操作，而synchronized的使用就会导致上下文的切换。

synchronized用的锁是存在Java对象头里的。

* 偏向锁：加锁和解锁不需要额外的消耗
* 轻量锁：竞争的线程不会阻塞
* 重量锁：线程竞争不使用自旋，不会消耗CPU

#### volatile

由于锁太笨重，会带来上下文的切换开销，杜宇解决内存可见性问题，Java还提供一种弱形式同步，就是使用volatile关键字，该关键字可以确保一个变量的更新和对其他线程马上可见。当一个变量被声明为一个volatile时，共享变量不会把值缓存到寄存器或者其他中，会直接更新到主内存中。

**特性**：可见性、原子性

**实现原则**

* Locak前缀指令会引起处理器缓存直接写回到内存
* 一个处理器的缓存回写到内存会导致其它处理器的缓存无效

及和synchronized的使用区别在于，一个在于使用在方法上，一个使用在变量上。

虽然volatile提供了可见性保证，但并不保证操作的原子性。

> 内存可见性问题：当一个线程对共享变量进行修改的时候，另一线程能直接获取到修改后的共享变量

使用场所：

* 写入变量值不依赖变量的当前值。
* 读写变量值时没有加锁。

#### Java中的CAS操作

CAS及Compare And Swap，其是JDK提供的非阻塞原子性操作。

* 经典ABA问题
  * 通过添加一个时间戳进行解决

## 锁

### 锁的内存语义

* 线程A释放一个锁，实际上是线程A向接下来要获取该锁的线程发送消息
* 线程B获取一个锁，实际上是线程B接收了某个线程发出的消息（在释放这个锁对共享变量进行修改的）消息
* 线程A释放锁，线程B接收到线程A释放的锁，实际上是线程A通过主内存向线程B发送消息。

### 乐观锁和悲观锁

#### 悲观锁

悲观锁的定义来源于数据库引入的名词，但在并发中也引入了此思想。

悲观锁指对外界修改持有保守态度，认为数据很容易被其它线程修改，所以在数据被处理前，先对数据加锁，并在整个数据处理对的过程中，使得数据处于锁定的状态。

#### 乐观锁

乐观锁是相对悲观锁来说的，认为数据在一般情况下不会造成冲突，所以在访问记录前不会加排它锁，而是在进行数据提交更新时，才会正式对数据冲突进行检测。

### 公平锁和非公平锁

**按照抢锁机制来划分**

公平锁表示线程获取锁的顺序是按照线程的请求顺序来进行的。先到先得。

ReentrantLock提供公平与非公平锁的实现：

```java
// 公平锁
ReentrantLock pairLock = new ReentrantLock（true）
// 非公平锁
ReentrantLock pariLock = new ReentrantLock（false）
```

在没有公平的需求下，尽量使用非公平锁，公平锁会带来性能开销。

#### 独享锁与共享锁

**按照锁被单个线程持有还是能被多个线程持有**

独占锁 = 悲观锁

共享锁 = 乐观锁


