# 一夜技术博客

## I/O:经典I/O、NIO、NIO2

#### 经典I/O(BIO)
经典I/O的场景是基于文件流的I/O
File对象针对文件做相关操作和RandomAccessFile对文件进行读写操作
InputStream和OuputStream是面向字节流的输入输出
Writer和Reader是面向字符集的输入输出
面向字符和字节的I/O的类基本是类似的

#### NIO
NIO是基于通道和缓冲区的形式进行数据处理的，NIO是双向的
NIO和组成Buffer和Channel

#### NIO2

## 事务、锁、死锁

### 锁

#### 乐观锁和悲观锁

乐观锁和悲观锁是预测并发访问数据发生冲突时使用的手段

悲观锁认为冲突是时刻存在的，通过排他方式阻止冲突的发生

悲观锁任务冲突是偶发的，先操作，发生冲突就采取重试的策略

## Java并发

### Java竞态条件

Java中的 **竞态条件** 是一种并发错误或问题，它是在您的程序中引入的，因为您的程序在多个线程同时并行执行  。当两个线程 在没有正确同步的同一对象上操作并且操作彼此交错 时，就会出现 **竞态条件**，临界区指：导致竞态条件发生的代码区

```java
@NotThreadSafe
public class LazyInitRace {
    private ExpensiveObject instance = null;
    public ExpensiveObject getInstance() {
        if (instance == null)
					instance = new ExpensiveObject();
				return instance;
    } 
}
```

多个线程同时执行单例的延迟加载时,会产生竞态，会发生返回的实例不是同一个实例的情况，如果这个单例存储重要数据，就会造成不一致的情况。这里的临界区就对单例的初始化判断

### Java并发基础

* 可见性

  一个线程多变量的修改对另外一个线程是不可见的，这是由于计算机CPU的多级缓存引起的，JMM的内存模型是对这一模型的简化

* 原子性

### Java线程安全

线程安全是指多线程情况下对java数据访问的一致性安全，

非线程安全是指在多线程情况下，对数据访问可能出现的不一致性问题。

#### 线程安全的实现

要实现线程安全，就要在多线程访问数据的情况下达到线程间数据的同步，或者避免多线程多数据的访问

> 使用Jvm提供的线程同步工具

* synchronized(代码块的同步)
* volatile修饰符(数据访问不经过寄存器，直接在主存级别)
* 显示锁
* 原子变量

> 避免多线程的数据访问修改，既线程数据间的共享

* 局部变量的使用
* 使用不可变变量和无状态对象
* threadLocal的使用

###



## Java线程和线程池

###Java线程

程序线性处理任务，无法充分利用计算机多核的能力。这时候可以用多进程能有效解决这个问题。高并发应用通常都会充分发挥多CPU的计算能力。**Java中线程的本质，其实就是操作系统中的线程**。线程模型的资源消耗更少，线程也是JAVA编程的标配，JVM也应用了很多，GC回收就是独立的线程进行的。

线程使用不单容易造成JVM虚拟机资源的耗尽，超过系统对底层的限制也会造成OOM的问题，频繁创建线程的开销也非常大。

```java
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
at java.lang.Thread.start0(Native Method)
at java.lang.Thread.start(Thread.java:714)
```

JVM可创建线程数受到分配的内存数影响，可以用-Xms -Xmx-Xmn 这几个参数调节，但也受到计算机关联配置的影响。所以任何不当的编程都会导致服务器的问题出现，自然java也为我们提供了有效的解决方案。

### Java线程池框架

#####Executors框架能直接调用静态构造方法创建配置好的线程池:

1.创建单线程线程池，能确保存在一个线程

```java
Executor singleThreadExecutor = Executors.newSingleThreadExecutor();
```

2.创建可缓存的线程池，任务超过线程数时，会回收空闲的线程

```java
Executor cachedThreadPool = Executors.newCachedThreadPool();
```

3.创建固定线程数的线程池，任务超过线程数，线程规模也不会增加

```java
Executor fixedThreadPool = Executors.newFixedThreadPool(10);
```

4.线程数等于CPU数，每个线程要处理分配队列中的任务，如果完成自己队列中的任务，那么它可以去其他线程中获取其他线程的任务去执行

```java
Executor workStealingPool = Executors.newWorkStealingPool();
```

5.创建固定长度的线程池，以时间调度的方式执行

```java
Executor scheduledThreadPool = Executors.newScheduledThreadPool(10);
```

6.创建单线程的线程池，以时间调度的方式执行

```java
Executor singleThreadScheduledExecutor = Executors.newSingleThreadScheduledExecutor();
```

##### ThreadPoolExecutor构造函数

虽然上述的预配置好的线程池能满足我们很大一部分需求，但可以通过 ThreadPoolExecutor 的构造函数进行定制实例化

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

构造函数参数：

* corePoolSize 线程池基本大小，即使线程空闲也会保存基本线程，除非allowCoreThreadTimeOut设置
* maximumPoolSize 线程池最大线程数，
* keepAliveTime 空闲线程存活时间，继超过核心线程数线程最大空闲存活时间
* unit 时间单位
* workQueue 任务队列（无界队列，有界队列，同步移交）
* threadFactory 线程构造工厂
* handler 线程饱和策略

重要参数介绍：

> workQueue 队列的选择对提交的任务管理非常重要，合适的队列选择关乎到应用的稳定

无界队列LinkedBlockingDeque，可以无限的增加提交的任务

有界队列ArrayBlockingQueue，队列填满之后会启动线程饱和策略处理	 

> threadFactory 是作为线程池构造线程调用的，默认框默认的线程工厂代码

```JAVA
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

> 饱和策略是用于处理有界队列被填满的情况

AbortPolicy 中止策略，默认的饱和策略，超过队列，会抛出异常

DiscardPolicy 抛弃策略，会抛弃无法执行的任务

DiscardOldestPolicy 抛弃最旧策略，会抛弃下一个将执行的任务，并且尝试重新提交新任务

CallerRunsPolicy 调用者模式, 队列饱和之后，该策略既不会丢弃任务和抛出异常，会转而给调用线程（通常是主线程）执行任务

#####ThreadPoolExecutor的钩子函数

可以通过ThreadPoolExecutor提供的钩子函数，来扩展ThreadPoolExecutor

* beforeExecute
* afterExecute
* terminated

### Java线程池原理

任务->工作线程->工作队列

> 线程池初始化

创建线程池，初始化基本参数，不做其他额外工作

> 线程池框架调用

添加工作任务，由于工作线程数小于核心线程数，会创建核心工作线程，如果已经超过工作线程限制，不再创建新的工作线程，将任务添加到工作队列，工作线程添加成功，开始执行工作线程

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```

> 工作线程运行

工作线程循环检查任务队列并执行

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                try {
                    task.run();
                    afterExecute(task, null);
                } catch (Throwable ex) {
                    afterExecute(task, ex);
                    throw ex;
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

工作线程获取任务逻辑

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();

        // Check if queue empty only if necessary.
        if (runStateAtLeast(c, SHUTDOWN)
            && (runStateAtLeast(c, STOP) || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

### Java调度线程池框架

ScheduledThreadPoolExecutor继承ThreadPoolExecutor了,借助线程池框架的功能，能很好代替Timer的功能权限，用来执行延迟任务和周期任务

| Timer                                            | ScheduledThreadPoolExecutor            |
| ------------------------------------------------ | -------------------------------------- |
| 单线程                                           | 多线程                                 |
| 单个任务执行时间影响其他任务调度                 | 多线程，不会影响                       |
| 基于绝对时间                                     | 基于相对时间                           |
| 一旦执行任务出现异常不会捕获，其他任务得不到执行 | 多线程，单个任务的执行不会影响其他线程 |

>  线程池初始化，使用DelayedWorkQueue作为任务队列

```java
public ScheduledThreadPoolExecutor(int corePoolSize,
                                       ThreadFactory threadFactory,
                                       RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue(), threadFactory, handler);
    }
```

> 线程池执行任务，添加到延迟队列中

```java
public ScheduledFuture<?> schedule(Runnable command,
                                   long delay,
                                   TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();
    RunnableScheduledFuture<Void> t = decorateTask(command,
        new ScheduledFutureTask<Void>(command, null,
                                      triggerTime(delay, unit),
                                      sequencer.getAndIncrement()));
    delayedExecute(t);
    return t;
}
```

> 获取队列元素

```java
public RunnableScheduledFuture<?> poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        RunnableScheduledFuture<?> first = queue[0];
        return (first == null || first.getDelay(NANOSECONDS) > 0)
            ? null
            : finishPoll(first);
    } finally {
        lock.unlock();
    }
}

public RunnableScheduledFuture<?> take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            RunnableScheduledFuture<?> first = queue[0];
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0L)
                    return finishPoll(first);
                first = null; // don't retain ref while waiting
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && queue[0] != null)
            available.signal();
        lock.unlock();
    }
}
```

### ForkJoinPool 线程框架

**forkJoinPool**的优势在于，可以充分利用多cpu，多核cpu的优势，把一个任务拆分成多个“小任务”，把多个“小任务”放到多个处理器核心上并行执行；当多个“小任务”执行完成之后，再将这些执行结果合并起来即可,forkJoinPool线程池为了提高任务的并行度和吞吐量做了非常多而且复杂的设计实现。

 `fork()` 和 `join()` 的作用：

- `fork()`：开启一个新线程（或是重用线程池内的空闲线程），将任务交给该线程处理。
- `join()`：等待该任务的处理线程处理完毕，获得返回值。



所谓**work-stealing模式**，即每个工作线程都会有自己的任务队列。当工作线程完成了自己所有的工作后，就会去“偷”别的工作线程的任务。



**ForkJoinPool与ThreadPoolExecutor区别：**

1.ForkJoinPool中的每个线程都会有一个队列，而ThreadPoolExecutor只有一个队列，并根据queue类型不同，细分出各种线程池

2.ForkJoinPool能够使用数量有限的线程来完成非常多的具有父子关系的任务,ThreadPoolExecutor中根本没有什么父子关系任务

3.ForkJoinPool在使用过程中，会创建大量的子任务，会进行大量的gc，但是ThreadPoolExecutor不需要，因此单线程（或者任务分配平均）

4.ForkJoinPool在多任务，且任务分配不均是有优势，但是在单线程或者任务分配均匀的情况下，效率没有ThreadPoolExecutor高，毕竟要进行大量gc子任务

 

ForkJoinPool在多线程情况下，能够实现工作窃取(Work Stealing)，在该线程池的每个线程中会维护一个队列来存放需要被执行的任务。当线程自身队列中的任务都执行完毕后，它会从别的线程中拿到未被执行的任务并帮助它执行。

ThreadPoolExecutor因为它其中的线程并不会关注每个任务之间任务量的差异。当执行任务量最小的任务的线程执行完毕后，它就会处于空闲的状态(Idle)，等待任务量最大的任务执行完毕。

因此多任务在多线程中分配不均时，ForkJoinPool效率高。





# Java知识

### Java标准库:集合类

### Java标准库:网络net

### Java标准库:并发current库

#### 并发同步机制

> AbstractQueuedSynchronizer

* ReentrantLock

* ReadWriteLock

* ReentrantReadWriteLock

* CountDownLatch

* CyclicBarrier

* CountedCompleter

* Semaphore

  Semaphore 是 synchronized 的加强版，作用是控制线程的并发数量

> StampedLock

#### 

####原子变量类

| 原子变量类             | 名称               | 底层结构 |
| ---------------------- | ------------------ | -------- |
| AtomicBoolean          | 布尔值原子变量     | Booean   |
| AtomicInteger          | 整数原子变量       | int      |
| AtomicIntegerArray     | 整形数组原子变量   | int[]    |
| AtomicLong             | 长整型原子变量     | long     |
| AtomicLongArray        | 长整型数组原子变量 | long[]   |
| AtomicReference        | 引用原子变量       | object   |
| AtomicReferenceArray   | 引用数组原子变量   | Object[] |
| AtomicStampedReference | 时间戳原子变量     | Pair<V>  |

基于CAS的实现方式



#### 线程安全集合

* ConcurrentHashMap
* ConcurrentLinkedDeque
* ConcurrentLinkedQueue
* ConcurrentSkipListMap
* ConcurrentSkipListSet
* CopyOnWriteArrayList
* CopyOnWriteArraySet



### Java标准库:工具util

### Java Spi, Spring Spi, Dubbo Spi

SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件
只要按照SPI的约定编写接口和对应实现类和配置文件，就能自动加载对应的服务提供实现

1. 在META-INF/services/ 目录中创建以接口全限定名命名的文件，该文件内容为API具体实现类的全限定名
2. 使用ServiceLoader类动态加载 META-INF 中的实现类
3. 如 SPI 的实现类为 Jar 则需要放在主程序 ClassPath 中
4. API 具体实现类必须有一个不带参数的构造方法

#### 服务接口类

```java
package com.sz.spi;
/**
 * @author sunze
 * @date 2020/10/9
 */
public interface Log {

    /**
     * 打印日志信息
     * @param str
     */
    void info(String str);
}
```
#### 实现类

```java
package com.sz.impl;
import com.sz.spi.Log;
/**
 * @author sunze
 * @date 2020/10/9
 */
public class TestLog implements Log {
    @Override
    public void info(String s) {
        System.out.println("Test:" + s);
    }
}
```
#### 实现类

```java
package com.sz.impl;
import com.sz.spi.Log;

/**
 * @author sunze
 * @date 2020/10/9
 */
public class DevLog implements Log {
    @Override
    public void info(String s) {
        System.out.println("Dev:" + s);
    }
}
```
#### 测试类

```java
package com.sz;
import com.sz.spi.Log;
import java.util.Iterator;
import java.util.ServiceLoader;

/**
 * @author sunze
 * @date 2020/10/9
 */
public class SpiTest {
    public static void main(String[] args) {
        ServiceLoader<Log> peoples = ServiceLoader.load(Log.class);
        Iterator<Log> iterator = peoples.iterator();
        while (iterator.hasNext()) {
            Log log = iterator.next();
            log.info("hellow wolrd");
        }
    }
}
```
运行结果：
Test:hellow wolrd
Dev:hellow wolrd

### Java原生调用

### Java内部类

java内部类是在类中声明了其他类，java内部类形式：

* 静态成员类
* 非静态成员类
* 局部类
* 匿名类

### Java范型

范型的特性集中在三个方面：

##### **范型类**

##### **范型方法**



##### 范型通配符

范型无界通配符和范型有界通配符

在特定情况下，我们可能并不关心范型具体的类型，它可以是某个确定的类型就行了，这时候可以用到通配符，不要用原生类型会造成没有编译期的检查导致潜在的运行错误。

> 应用例子

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

##### 范型实现-擦除

### Java枚举类

java枚举类提供了类空间的枚举单例

### JVM性能调优工具:jstat、jmap、jstack、jinfo

### Java并发

并发数据结构：阻塞型数据结构和非阻塞型数据结构

### Jvm开放的接口

# 云原生技术

云原生让分布式服务端系统成为可能，能简单的管控计算资源，只需要将计算机加入k8s作为集群node

| 单机系统         | k8s分布式系统      |
| ---------------- | ------------------ |
| 进程             | 应用系统           |
| 单应用           | 分布式             |
| 进程间通信       | 网络间通信         |
| 资源管控能力有限 | 统一的资源管控     |
| 文件系统         | PV/PVC             |
| 计算能力有限     | 计算能力理论上无限 |
| 单机硬件         | 集群机器           |

### Kubernetes

k8s解决了分布式应用的管理问题

#### Kubernetes组成

> Master节点组件控制节点，主节点，负责管理集群状态
* etcd配置存储
* API服务器
* 调度器
* 控制器管理器
> Node节点的组件, 运行在工作node上
* Kubelete 管理pod
* kube-proxy 网络通信和负载
* 运行容器 
> 附加组件
* Kubernetes的DNS服务器
* 控制台
* Ingress控制器
* 监控工具
* 容器网络插件
Kubernetes架构图
![Image](src)

> pod的生命周期

- Pending：表示pod已经被同意创建，正在等待kube-scheduler选择合适的节点创建，一般是在准备镜像；
- Running：表示pod中所有的容器已经被创建，并且至少有一个容器正在运行或者是正在启动或者是正在重启；
- Succeeded：表示所有容器已经成功终止，并且不会再启动；
- Failed：表示pod中所有容器都是非0（不正常）状态退出；
- Unknown：表示无法读取Pod状态，通常是kube-controller-manager无法与Pod通信

#### Kubernetes资源限制

> Qos服务质量等级
>
> * (低) BestEffort 最大努力
> * (中) Burstable 
> * (高) Guaranteed保证

> ResourceQuota
>
> 命名空间资源配额

pod资源使用超出node资源时， 会对低服务质量等级的容器内进程进行kill操作

#### Kubernetes核心理念

> kubernetes的pod健康检查(liveness probees)
* HTTP GET获取POD状态
* TCP Socket状态检查
* Exec 命令方式 
> kubernetes的pod准备检查(readiness probe)
* HTTP GET获取POD状态
* TCP Socket状态检查
* Exec 命令方式

> kubernetes的services
k8s的pod的ip是可变不固定的，多pod的ip也不同，需要service作为固定唯一的访问入口
集群内部service访问:添加selector, 环境变量和FQDN
集群外部service访问:不指定selector, 创建endpoints资源关联外部集群外部服务ip:port

> kubernetes的外网访问机制
k8s尽开放了pod间的相互访问，pod的ip对外部是不可见的，外网访问需要固定的ip入口，外网访问k8s的机制有一下几种
* NodePort(重定向外部请求包到service,可以通过node的ip访问service)
* LoadBalancer(NodePort的扩展类型) 
* Ingress(HTTP网络层) 

> kubernetes的服务访问异常排查
1. 集群内部访问服务集群IP
2. 确保pod的健康检查是OK的
3. 确认pod是服务的一部分，检查服务的endpoint
4. 通过FQDN访问确保是正确的
5. 确认你连接的是服务的port而部署目标port

> kubernetes的无状态控制器
* ReplicationController[已废弃]
* ReplicaSets(Pod模版, 标签选择，Pod数)
* DaemonSet(一个Node一个Pod)
* Job/CronJob
* Deployment
> kubernetes的有状态控制器
> kubernetes的自定义控制器

> kubernetes的volumn
用来挂载持久化的数据
* emptyDir在pod的容器中共享，pod删除后一起消失
* hostpath在node节点上的目录
* gitRepo获取git内容
* PVC屏蔽了PV的多样性，提供了一致的接口
动态数据传递, 命令行和环境变量无法避免去修改k8s的pod资源文件, 配置和密码将动态的数据从镜像抽离出来，减少不必要的镜像重构和维护
* 命令行command和args
* 环境变量env
* ConfigMap
ConfigMap用键值的方式保存配置, 可以通过字面量设置，可以读取文件设置, 也可以读取文件夹设置
ConfigMap可以作为容器环境变量读取,也可以作为文件挂载到容器 
ConfigMap的修改会更新到容器里面，避免了容器的重新启动
* Secrets
用来存储敏感信息，不会落磁盘文件

> kubernetes的应用
kubernetes用四种资源组成一个应用（workloads, loadbalance, service, volumes)

### Docker
####ENTRYPOINT和CMD的区别
ENTRYPOINT会在容器启动时执行，不会被覆盖
CMD在启动运行容器时执行，可以在docker run时覆盖执行命令
####ADD和COPY的区别
功能都是将文件添加到镜像，ADD会多其他功能，针对tar的压缩文件会解压, 从url拷贝文件到镜像中

#### Docker工具

skopeo， kaniko， s2i

# 分布式应用框架

### Dubbo
### SpringCloud
### Spring,SpringMvc,SpringBoot
作为知名的MVC框架，随着前后端分离的持续推进，越来越多应用不再使用MVC的模式, 主要原因是模版渲染不再被使用

# 数据库

数据库作为持久方式，是提供有状态的服务，数据库的底层还是用文件的形式实现

### mysql的事务隔离级别

| 隔离级别                     | 脏读（Dirty Read） | 不可重复读（NonRepeatable Read） | 幻读（Phantom Read） |
| :--------------------------- | :----------------- | :------------------------------- | :------------------- |
| 未提交读（Read uncommitted） | 可能               | 可能                             | 可能                 |
| 已提交读（Read committed）   | 不可能             | 可能                             | 可能                 |
| 可重复读（Repeatable read）  | 不可能             | 不可能                           | 可能                 |
| 可串行化（Serializable ）    | 不可能             | 不可能                           | 不可能               |

Read uncommitted 提交可能读到脏数据，生产推荐使用

Serializable 级别对性能影响很大，特定场景才使用



# 中间件技术

对任何中间件的引入都要谨慎，一定程度上破坏来应用的无状态，同时造成了应用复杂度的上升，中间件的维护成本。
中间件技术本质是用来减小代码复杂度和解耦的作用



### 缓存
#### 分布式缓存Redis
#### 应用本地缓存
### 索引
### 消息中间件

> 常见的消息中间件

> 如何防止消息丢失？

### 配置中心
配置中心将常变的数据从代码中抽离出来， 不再依赖于应用的重新发布，但也会造成配置的更新不再和应用发布同步, 
可以通过配置新的键值来同步发布配置

### 数据库中间件
### 分布式任务框架

quartz, X-Job, airflow等都支持分布式调用





### 分布式存储方案

### 定时调用器

#### Cron基本知识点介绍

Cron表达式是一个字符串，字符串以5或6个空格隔开，分为6或7个域，每一个域代表一个含义，Cron有如下两种语法格式：

1. Seconds Minutes Hours DayofMonth Month DayofWeek Year
2. *Seconds Minutes Hours DayofMonth Month DayofWeek*

### 唯一ID生成服务
### 告警系统
Minlo

## 代码库

### JSON和XML解析

> json解析
>
> * fastjson
> * gson

> xml解析
>
> * dom4j（dom4j解析快，支持xpath，使用方面推荐使用）
> * dom（通用的解析方案，逻辑简单，编写麻烦）

## 接入层网关



## 网络协议
### HTTP/HTTP2/HTTP3

### 文件下载协议

# 设计模式

设计模式更多的是对过去编程经验的总结，形成的一套抽象的语言系统描述背后的设计思想，设计模式提高了维护效率但不会提高运行效率,本质上是对代码的组织维护的提升,加强整个工程的稳定性,是一种潜在的编程约定让其他人不会破坏这种约定
### 开放封闭原则OCP（对扩展开放，对修改封闭）
是所有面向对象原则的核心, 软件设计本身所追求的目标就是封装变化、降低耦合，而开放封闭原则正是对这一目标的最直接体现.
### 代理模式
代理模式被使用来实现对象的访问控制，对调用方隐藏真正的对象

> 代理模式的实现方式

### 装饰者模式
装饰者模式实现了对象的增强, 和基础对象拥有一样的方法，持有基础对象，同名方法在调用基础对象后会做额外的行为，Java I/O包就使用了这个设计模式。

### 门面模式



### 组合模式

将依赖对象组合在属性中，要比继承有更好的封装，继承破坏了面向对象的封装特性

### -- 行为模式 -- 

#### 状态模式

在状态模式（State Pattern）中，类的行为是基于它的状态改变的。这种类型的设计模式属于行为型模式。

在状态模式中，我们创建表示各种状态的对象和一个行为随着状态对象改变而改变的 context 对象。

#### 策略模式

在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。lamda表达式的作为借口策略提供不同的行为结果



## 编码技巧

####Builder构建

在类构建参数很多的情况下，传参会成为潜在的问题，调用set方法的构建无法确保一致性，build模式能避免这种情况，通过编写类的Builder内部类来实现。这种技巧在许多工程项目中被用到

```java
public class People {

    private String name;
    private String state;
    private int age;
    private double height;

    public static class Builder {

        private String name = "";
        private String state = "";
        private int age = 0;
        private double height = 0.0;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder state(String state) {
            this.state = state;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder height(double height) {
            this.height = height;
            return this;
        }

        public People build() {
            return new People(this);
        }

    }

    private People (Builder builder) {
        this.name = builder.name;
        this.state = builder.state;
        this.age = builder.age;
        this.height = builder.height;
    }

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", state='" + state + '\'' +
                ", age=" + age +
                ", height=" + height +
                '}';
    }

    public static Builder builder() {
        return new Builder();
    }

    public static void main(String[] args) {
        People people = People.builder()
                .age(19)
                .state("china")
                .name("王飞")
                .height(180.5)
                .build();
        System.out.println(people);
    }
}
```

#### 依赖注入

在组合对象时， 不建议直接构建所依赖的对象， 这会使依赖对象难以测试，通常通过构建函数注入的方式，将所依赖对象作为参数注入，依赖注入在spring等框架都已经实现，并广泛使用





## 性能优化
性能优化要从测试，监控，分析，调优三个方面入手

#### 基准测试

程序的性能测试有多种工具提供使用

faban





#### 监控指标
> CPU使用率（用户时间和系统时间）
>
> 因为我们强调的是应用性能，所以系统态CPU使用率越低越好
>
> 网络IO
> 磁盘IO,空间
> 空间的使用情况
> 内存使用率
> 锁竞争



## 持续集成

持续集成的应用构成

> Git等代码管理工具

> 开发IDE静态规范检查

> CICD流水线

> 制品管理系统

# 编程语言

### Java应用启动过程
1. 解析命令行选项
2. 设置堆的大小和JIT编译器
3. 设置环境变量D_LIBRARY_PATH和CLASSPATH
4. JAR的manifest查找Main-Class或者从命令行读取Main-Class
5. 创建HotSpot VM线程
6. 加载JAVA Main-Class
7. HotSpot VM调用Java main和命令行参数传递

###java程序的调优和优化-JVM启动参数
java应启动，是通过启动JRE环境，加载指定的类，调用类的main函数，args的第一个参数通常是制定加载的类，或者如果-jar参数存在，就是JAR包的名称：
    public static void main(String[] args)
java命令参数支持许多种参数，可以分为一下几种类别：

- 标准参数
- 非标准参数
- 高级运行时参数
- 高级编译参数
- 高级服务参数
- 高级垃圾回收参数

标准参数(是最经常使用的参数，所有的JVM实现都会支持)
-verbose:jni 打印原声方法调用信息学
-verbose:gc 打印gc信息
-verbose:class 打印类信息
-jar filename 执行jar包程序
-help 显示帮助信息
-Dproperty=value 设置属性值
-server 服务端的模式启动
-client 客户端的模式启动

非标准参数(非标准参数是给HotSpot虚拟机使用的)

-X 显示所有非标准参数

-Xmnsize 设置年轻代大小

-Xmssize 设置堆初始化大小

-Xmxsize 设置分配内存最大值

-Xsssize 设置线程栈大小

-Xnoclassgc 不进行垃圾回收

-Xprof 打印运行记录，在开发环境有用

-XshowSettings:category 显示配置项*

高级运行参数

-XX:ThreadStackSize=size 设置线程栈大小

高级编译参数

高级垃圾回收参数

# 技术理念

### 软件应用

软件应用是不稳定的，类比现实生活中的建筑，硬件有老化的缺点，但一定时间范围内是稳定的，做好定期维护就好，而软件应用依赖于硬件环境，处理硬件问题要考虑外，主要自身是一定时间范围内也无法确保是稳定的，但某种意义上是不存在老化的问题，重新启动能解决绝大部分问题，频繁的迭代也破坏了自身的稳定性，传统软件的稳定性要比互联网应用好是因为传统软件一半只依赖系统环境而互联网应用往往都是分布式应用，依赖是不可靠的，稳定性有时候很难保障，为了保障整个系统的健壮，分布式系统更类似于人体，一个个应用更类似于细胞，往往身体组织是由成千上万统一类别细胞组成， 细胞不断消亡产生，分布式应用也是如此，不能一个应用的可用性，但能基本保证一组应用的可用性，软件应用的发展越来越朝这方面发展，保证整体的可用性，不保证个体的可用性。

### 分布式和高并发

分布式和高并发没有必然关系， 高并发在任何场景都会涉及，只要用户端同时批量请求，服务端会多线程或进程处理用户请求，在多核服务端处理下，必然会有一致性问题，
而分布式是用来面对大流量请求的一种架构设计，是建立在网络通信基础上的，带来伸缩性上的方便的同时增加了系统的复杂程度,分布式也是通常用来应对大流量的一种基数方案，
大流量下必然会有高并发的问题，就导致了分布式和高并发通常是同时出现

### 代码编写

代码编写是件掌握很容易，精进很难的东西，架构能解决一部分大量代码的问题，却解决不了细节问题，就像你房地产将一栋房子搭建起来，达到稳固的要求，每间房间的装修却差的要命，代码编写是一项细致活，是你的美学品味，非常难提升，编写大量的代码一点不难， 难得任何人永远是维护和修改代码，如果一个功能过了一个月让人修改，通过阅读源码，做到快速理解，代码层次要像规划良好城市道路那样事清晰，而不是揉杂一起的线团让人看了头麻。





## Linux知识

Linux中的虚拟设备

> /dev/null

> /dev/zero

> /dev/random

> /dev/urandom



## 系统架构

* 易开发，易部署，易维护作为评判指标
* 系统架构很重要的一点就是依赖清晰， 耦合性低



## web架构
web架构演化至今已经形成了多层架构的模式，网络代理在其中扮演了非常重要的角色，客户端进入到服务端的网络，到请求到达真正的服务器处理之间可能经过多层的网络代理

#### 并发问题
> 文件锁
解决进程间对同一资源的占用访问问题



## 开发注意事项
* 数字的更新操作不能用set操作，应该用原子性的incr操作，mysql和redis都有使用场景 

* lombok的使用是对代码有很强的入侵，隐藏代码的语法糖，而且必须添加IDE插件和相关依赖, 目前IDE都提供相关函数自动生成的工具，编写工作没我们想的那么繁重，
  对lombok的使用其实应该慎重
  
* mybatis-plus这个插件不建议使用，不是事实的规范，增加维护成本，造成编写的代码及其随意

* 对不熟悉的库谨慎使用

* 开发一个很重要的目标是可扩展性和延展性，之后的大多数更改只需要做微笑的调整就能适配才是高质量的开发

  



## Java开发注意事项
* （避免空指针异常）字符串常量作为equal调用方，能避免nullpoint
* （私有化构造函数）像Arrays, Collections， 对不可实例化的类，通常作为工具类，需要将构造函数设置为私有函数
* （避免不必要的自动装箱拆箱）不必要的自动装箱拆箱会频繁构造对象，函数内部能使用原始类型计算就使用，能有效减少原始类型的装箱操作
* （避免内存泄漏）java虽然会自动回收对象但是对不再使用的对象如果不消除，也会造成内存泄漏
* （尽量使用不可变对象）不可变对象能有效保持程序的健壮性
* Pojo对象避免默认值，预设默认值其他开发人员根本不知道
* 返回空集合或数组而不是对象，后者容易导致调用方空指针
* 要确保自己写的程序没有明显的性能缺陷，比如循环中查询数据库等耗费资源的操作
* 开发需要注意程序崩溃了，会有什么影响，数据丢失了会有什么影响
* 编写类也要符合单一职责， 类太大再复用方面会有很大问题，无特殊场景不建议内部类的使用



### Java开发问题和解决

> JVM的OOM



## Java开发工具
> IDEA开发插件
- 阿里巴巴编码规范插件安装
- lombok插件配置
- mybatis插件Free-idea-mybatis安装
- Rainbow Brackets （括号高亮）插件安装






### 个人情况
- 系统：linux、kubernetes
- 语言：Java、PHP、Goland、Python
- 数据库: mysql
- 框架: Dubbo

