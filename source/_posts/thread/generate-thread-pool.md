---
title: 创建线程池的五种方式
date: 2022-03-21 22:16:43
tags: [ThreadPool]
categories: [多线程]
---

茴字到底有几个写法？ oh... 不， 是线程池到底有几种创建方式
<!--more-->


# 为什么用线程池

- 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行
- 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控

# 有哪几种创建线程池的方式

一共五种，其实最终只是一种，都是利用ThreadPoolExecutors 创建

分别是

```java
Executors.newCachedThreadPool()

Executors.newFixedThreadPool(int nThreads)

Executors.newSingleThreadExecutor()// 默认一个线程

Executors.newScheduledThreadPool(int poolSize)  
```

以及

```java
new ThreadPoolExecutors(

	int corePoolSize,

	int maximumPoolSize,

	long keepAlive,

	TimeUnit timeunitEnum,

	BlockingQueue<T> workQueue,

	ThreadFactory threadFactory,

	RejectedExecutionHandler abortpolicy);
```

# 线程池该怎么创建，有哪些参数

![thread-pool-execute.png](thread-pool-execute.png)

```
判断核心线程数是否已满，核心线程数大小和**corePoolSize**参数有关，未满则创建线程执行任务

若核心线程池已满，判断队列是否满，队列是否满和**workQueue**参数有关，若未满则加入队列中

若队列已满，判断线程池是否已满，线程池是否已满和**maximumPoolSize**参数有关，若未满创建线程执行任务

若线程池已满，则采用拒绝策略处理无法执执行的任务，拒绝策略和**handler**参数有关
```

# 为什么不用Executors 创建线程

**newFixedThreadPool ，newCachedThreadPool,  newSingleThreadExecutor以及newScheduledThreadPool 都可能因为资源耗尽导致 OOM 问题,**出现**`java.lang.OutOfMemoryError: unable to create new native thread`.**

## **newFixedThreadPool**

看**newFixedThreadPool**源码，核心线程数和最大线程数相同，不难发现线程池的工作队列直接new了一个LinkedBlockingQueue，**而默认构造方法的 LinkedBlockingQueue 是一个 Integer.MAX_VALUE 长度的队列，可以认为是无界的，如果任务较多并且执行较慢的话，队列可能会快速积压，撑爆内存导致 OOM**

```java
public static ExecutorService **newFixedThreadPool**(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  ****0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}

public LinkedBlockingQueue() {

	this(Integer.MAX_VALUE);  // 默认无界队列

}
```

**newFixedThreadPool** 虽然线程数量是固定的， 但是队列是无界的， 如果任务堆积很容易造成OOM,

## **newCachedThreadPool**

看**newCachedThreadPool**源码

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```

**这种线程池的最大线程数是 Integer.MAX_VALUE，可以认为是没有上限的，而其工作队列**
**SynchronousQueue 是一个没有存储空间的阻塞队列，**这意味着，只要有请求到来，就必须找到一条工作线程来处理，如果当前没有空闲的线程就再创建一条新的。线程有60s的存活时间即实现了一个带缓存的线程池

## **newSingleThreadExecutor**

只有一个核心线程，最大线程数也只允许为1

当任务提交时会先创建一个线程去执行任务，如果超过核心线程梳数量，将会放入队列中，同样也是一个无界阻塞队列，资源优先时也会引起OOM

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

## **newSingleThreadScheduledExecutor** 周期性线程池

```java

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
}

```

**总结：**

不建议使用 Executors 提供四种创建线程池的方法，原因如下：

- newCachedPool, newSchedulePool 允许创建的线程数量是Integer.MAX_VALUE，可能会创建大量线程导致OOM

- newFixedPool, newSinglePool 因为使用的LinkedBlockQueue 允许的最大长度是Integer.MAX_VALUE 可能会堆积大量请求导致OOM

我们需要根据自己的场景、并发情况来评估线程池的几个核心参数，包括核心线程数、最大线程数、线程回收策略、工作队列的类型，以及拒绝策略，确保线程池的工作行为符合需求，一般都需要设置有界的工作队列和可控的线程数。

任何时候，都应该为自定义线程池指定有意义的名称，以方便排查问题。当出现线程数量暴增、线程死锁、线程占用大量 CPU、线程执行出现异常等问题时，我们往往会抓取线程栈。此时，有意义的线程名称，就可以方便我们定位问题