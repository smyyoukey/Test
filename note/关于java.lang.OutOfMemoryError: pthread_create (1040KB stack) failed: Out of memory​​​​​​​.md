在后台的错误日志中发现了
### Throwing  OutOfMemoryError  "pthread_create  (1040KB  stack)  failed:  Out  of  memory"
搜索看到的结果说，
#### 1.常规的 OOM 一般发生在堆内存中；
##### 2.而发生 OOM 不一定是因为设备的内存(堆内存)不够用，也就是说内存宽裕也可能发生 OOM，比如栈内存溢出……

怎么就栈内存溢出了？而且还是线程不够用了？我没对栈内存做什么处理呀

后来查了pthread_create相关关键词，发现好像确实是和线程数有关，特别是华为部分手机线程数阈值较低容易出现这个问题，但怎么会发生线程爆满呢？而且之前版本为啥没这个问题？?

创建大量的空线程（不做任何事情，直接sleep）

实验预期：
当线程数（可以在/proc/pid/status中的threads项实时查看）超过/proc/sys/kernel/threads-max中规定的上限时产生OOM崩溃

实验结果：

在Android7.0及以上的华为手机（EmotionUI_5.0及以上）的手机产生OOM，这些手机的线程数限制都很小(应该是华为rom特意修改的limits)，每个进程只允许最大同时开500个线程，因此很容易复现了。OOM时错误信息如下：
```java
W libc    : pthread_create failed: clone failed: Out of memory
W art     : Throwing OutOfMemoryError "pthread_create (1040KB stack) failed: Out of memory"
E AndroidRuntime: FATAL EXCEPTION: main
                  Process: com.netease.demo.oom, PID: 4973
                  java.lang.OutOfMemoryError: pthread_create (1040KB stack) failed: Out of memory
                      at java.lang.Thread.nativeCreate(Native Method)
                      at java.lang.Thread.start(Thread.java:745)
                      ......
                      ```
 可以看出错误信息与我们线上遇到的OOM二吻合："pthread_create (1040KB stack) failed: Out of memory"

#### 最后可以确认是线程数超限，即proc/pid/status中记录的线程数（threads项）突破/proc/sys/kernel/threads-max中规定的最大线程数。可能的发生场景有：
app内多线程使用不合理，如多个不共享线程池的OKhttpclient等等

查找相关资料，有说到线程创建使用的不是堆内存，而是实际物理内存。由于没有对应用的线程数资源进行监控，同时也没有对当时的内存资源进行监控，所以只能根据应用的日志进行分析。通过查看异常栈，发现是某个线程池在开启一个新的线程时会报出该异常，并且该线程池使用的是无界线程池，也就是说，当执行的任务一直没有完成时，线程池会持续不断的开启新的线程。

#### 那么之后的分析过程就是查找如何导致的创建大量线程，在应用中对线程的使用其实很简单，主要有以下几点：

1.有需要耗时操作的逻辑，直接使用new Thread（）开启线程，某些在过程中或结束时发handler或eventbus通知主线程;

2.需要周期性执行的耗时i/o操作使用单例线程池，某些在服务中启动的任务在Task实例中初始化线程池;

3.使用Rxjava进行并发处理操作。（并行和并发不同但在这里不做讨论）

再次在网上查找问题发现很多博客或论坛中遇到这个问题基本上都是使用了一些特定的第三方框架，比如说Volley，OkHttp + Retrofit，Rxjava等，更有博主提到 “OkHttpClient 和 Retrofit 写的不是单例，每次请求都重新创建了一个新的，每次创建的OkHttpClient 耗用大量资源，而且会创建大量的线程无论是用 Volley 还是 OkHttp + Retrofit，都要避免每次请求创建新的 RequestQueue, OkHttpClient 或 Retrofit，要写成单例模式，减少重复创建而造成的资源浪费。”

#### 结合之前问题出现的版本号和一些分析，定位到应该是Rxjava的使用导致的线程创建问题。

```java

本人使用的Rxjava2 (版本是2.2.2)；线程调用器使用的Scheduler.io()通过查看
    public static Scheduler io() {
    return RxJavaPlugins.onIoScheduler(IO);
    }
    -->
    public static Scheduler onIoScheduler(@NonNull Scheduler defaultScheduler) {
    Function<? super Scheduler, ? extends Scheduler> f = onIoHandler;
    if (f == null) {
        return defaultScheduler;
    }
    return apply(f, defaultScheduler);
    }
    -->默认的调度器就是IO:
     IO = RxJavaPlugins.initIoScheduler(new IOTask());
    -->通过IOTask():
    static final class IoHolder {
    static final Scheduler DEFAULT = new IoScheduler();
    }
    -->IoScheduler()内部：
    CachedWorkerPool(long keepAliveTime, TimeUnit unit, ThreadFactory threadFactory) {
        this.keepAliveTime = unit != null ? unit.toNanos(keepAliveTime) : 0L;
        this.expiringWorkerQueue = new ConcurrentLinkedQueue<ThreadWorker>();
        this.allWorkers = new CompositeDisposable();
        this.threadFactory = threadFactory;

        ScheduledExecutorService evictor = null;
        Future<?> task = null;
        if (unit != null) {
            evictor = Executors.newScheduledThreadPool(1, EVICTOR_THREAD_FACTORY);
            task = evictor.scheduleWithFixedDelay(this, this.keepAliveTime, this.keepAliveTime, TimeUnit.NANOSECONDS);
        }
        evictorService = evictor;
        evictorTask = task;
    }
    -->可以发现是通过Executors.newScheduledThreadPool(1, EVICTOR_THREAD_FACTORY)来构造线程池；
    -->：
    public ScheduledThreadPoolExecutor(int corePoolSize,
                                   ThreadFactory threadFactory) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue(), threadFactory);
    }
    -->可以看到最大线程数是无限大的；默认保活时间是10秒。
    -->所以，当你频繁使用Rxjava2的Scheduler.io()来执行任务，
    可能会出现栈内存溢出的情况。特别是一些华为手机。通过Android Profilter观察CPU使用情况，就可以发现大量的线程被创建，
    而且没有被及时杀死。
```

#### 该问题的解决方案：
```java
自定义Schedulers的线程池，在频繁使用Rxjava2的时候仅使用单个调用度的实例。 例如：

    if (scheduler==null){
    scheduler = Schedulers.from(Executors.newFixedThreadPool(10));
    }
    observable.subscribeOn(scheduler)
                    .unsubscribeOn(scheduler)
                    .observeOn(AndroidSchedulers.mainThread())
                    ```

然而我遇到的并不是这个问题（但是差不多）。
### 而是subscribe（）中发射事件的实参 ObservableEmitter在完成之后一定要发射onComplete（）事件。disposable对象最好也在完成之后dispose（）。dispose不会停止异步事件而是只会会解除订阅。
