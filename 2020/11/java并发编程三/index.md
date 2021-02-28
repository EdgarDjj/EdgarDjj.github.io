# Java并发编程三


# Java并发编程三

## 线程池的创建

### 一、newFixedThreadPool

创建定长线程池，每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程数量不再变化，当线程发生错误结束时，线程池会补充一个新的线程。

```java
package Advance.concurrent_programming.thread_pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/11/18
 **/
public class TestNewFixedThreadPool {
    public static void main(String[] args) throws InterruptedException {
        // 创建一个线程数目为3的newFixedThreadPool
        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
        for (int i = 0; i < 6; i++) {
            final int index = i;
            fixedThreadPool.execute(() ->{
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " index : " + index);

            });
        }
        Thread.sleep(2000);
        System.out.println("after 2 seconds");
        // 关闭线程池后，已提交的任务仍然会执行完
        fixedThreadPool.shutdown();
    }
}

```

```
执行结果：
pool-1-thread-2 index : 1
pool-1-thread-1 index : 0
pool-1-thread-3 index : 2
after 2 seconds
pool-1-thread-1 index : 4
pool-1-thread-3 index : 5
pool-1-thread-2 index : 3
```

### 二、newCachedThreadPool

创建可缓存的线程池，如果线程池的容量超过了任务数，自动回收空闲线程，任务增加时可以自动添加新的线程，线程池的容量不限制。

```java
package Advance.concurrent_programming.thread_pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/11/18
 **/
public class TestNewCachedThreadPool {
    public static void main(String[] args) throws InterruptedException {
        //创建可缓存的线程池，如果线程池的容量超过了任务数，
        //自动回收空闲线程，任务增加时可以自动添加新线程，线程池的容量不限制
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        for (int i = 0; i < 6; i++) {
            final int index = i;
            cachedThreadPool.execute(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " index : " + index);
            });
        }
        Thread.sleep(2000);
        System.out.println("after 2 seconds");
        cachedThreadPool.shutdown();
    }
}
```

```
运行结果：
pool-1-thread-1 index : 0
pool-1-thread-3 index : 2
pool-1-thread-5 index : 4
pool-1-thread-2 index : 1
pool-1-thread-4 index : 3
pool-1-thread-6 index : 5
after 2 seconds
```

可以发现创建的线程数目与任务数目相同。

### 三、newScheduledThreadPool

创建定长线程池，可执行周期性的任务

```java
package Advance.concurrent_programming.thread_pool;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

/**
 * Description:
 *
 * @author:edgarding
 * @date:2020/11/18
 **/
public class TestNewScheduledThreadPool {
    public static void main(String[] args) throws InterruptedException {
        // 创建定长线程池，可执行周期性的任务
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(3);
        for (int i = 0; i < 3; i++) {
            final int index = i;
            //scheduleWithFixedDelay 固定的延迟时间执行任务；
            // scheduleAtFixedRate 固定的频率执行任务
            scheduledExecutorService.scheduleWithFixedDelay(() -> {
                System.out.println(Thread.currentThread().getName() + " index: " + index);
            }, 0, 3, TimeUnit.SECONDS);
        }
        Thread.sleep(4000);
        System.out.println("after 4 second");
        scheduledExecutorService.shutdown();
    }
}
```

```

运行结果：
pool-1-thread-1 index: 0
pool-1-thread-3 index: 2
pool-1-thread-2 index: 1
pool-1-thread-1 index: 0
pool-1-thread-3 index: 2
pool-1-thread-2 index: 1
after 4 second
```

### 四、newSingleThreadExecutor

创建单线程的线程池，线程异常结束，会创建一个新的线程，能确保任务按提交顺序执行。能确保任务在队列按照串行来执行。

```java
package Advance.concurrent_programming.thread_pool;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Description:
 * 测试单线程的线程池
 * @author:edgarding
 * @date:2020/11/18
 **/
public class TestNewSingleThreadExecutor {

    public static void main(String[] args) {
        //单线程的线程池，线程异常结束，会创建一个新的线程，能确保任务按提交顺序执行
        ExecutorService singleThreadPool = Executors.newSingleThreadExecutor();

        //提交 3 个任务
        for (int i = 0; i <3; i++) {
            final int index = i;
            singleThreadPool.execute(() -> {

                //执行第二个任务时，报错，测试线程池会创建新的线程执行任务三
                if (index == 1) {
                    throw new RuntimeException("线程执行出现异常");
                }

                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " index:" + index);
            });
        }

        try {
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("4秒后...");

        singleThreadPool.shutdown();
    }
}

```

```
运行结果：
pool-1-thread-1 index:0
Exception in thread "pool-1-thread-1" java.lang.RuntimeException: 线程执行出现异常
	at Advance.concurrent_programming.thread_pool.TestNewSingleThreadExecutor.lambda$main$0(TestNewSingleThreadExecutor.java:25)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
4秒后...
pool-1-thread-2 index:2
```


