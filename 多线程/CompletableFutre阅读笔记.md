# CompletableFutre学习笔记
<!-- TOC -->

- [CompletableFutre学习笔记](#completablefutre学习笔记)
  - [源码解析](#源码解析)
  - [原理](#原理)
    - [supplyAsync](#supplyasync)
    - [runAsync](#runasync)
    - [thenAccept](#thenaccept)
    - [thenApply](#thenapply)
    - [thenCombine](#thencombine)
    - [thenRun](#thenrun)
  - [性能测试对比](#性能测试对比)
    - [性能测试1](#性能测试1)
    - [性能测试2](#性能测试2)

<!-- /TOC -->
    - [性能测试1](#性能测试1)
    - [性能测试2](#性能测试2)

<!-- /TOC -->


在美团的文章中提到了为什么要用 CompletableFuture，而不是之前的线程池的方式，原因就是使用线程池会带来的线程的切换导致CPU的上下文切换从而影响性能。但是这篇文章又没有说明线程池的方式是如何影响了性能，CPU的上下文切换所带来的CPU上下文切换的应该如何衡量，所以这里可以看一下这篇文章-[上下文切换开销](https://www.hitzhangjie.pro/libmill-book/basics/context-switching-cost.html)。

第二个概念就是文章中提到了商家API作为提供给商家信息查询的流量入口，承担了向下游获取数据的功能。所以是IO密集型，那么什么是IO密集型呢？可以参考一下这篇文章[What do the terms "CPU bound" and "I/O bound" mean?](https://stackoverflow.com/questions/868568/what-do-the-terms-cpu-bound-and-i-o-bound-mean)

```
A program is CPU bound if it would go faster if the CPU were faster, i.e. it spends the majority of its time simply using the CPU (doing calculations). A program that computes new digits of π will typically be CPU-bound, it's just crunching numbers.

A program is I/O bound if it would go faster if the I/O subsystem was faster. Which exact I/O system is meant can vary; I typically associate it with the disk, but of course, networking or communication, in general, is common too. A program that looks through a huge file for some data might become I/O bound since the bottleneck is then the reading of the data from disk (actually, this example is perhaps kind of old-fashioned these days with hundreds of MB/s coming in from SSDs).
```
这篇文章中提到的就是“而商家端 API 服务是流量入口，所有商家端流量都会由其调度、聚合，对外面向商家提供功能接口，对内调度各个下游服务获取数据进行聚合，具有鲜明的 I/O 密集型（I/O Bound）特点”，其实目前不管是商店还是推荐都有这个问题，之前作为流量的入口，然后商店的功能现在是由推荐承接的。那么也就是说我其实要面临的是一个大流量场景。针对这种大流量多流量整合的场景，那么就和美团描述的是一样的了。但是我们采用的方式是多线程调用，而美团选择了CompletableFuture调用的方式，我们可以看一下美团的CompletableFuture给美团接口性能带来了多少的业务提升。

美团一开始使用了线程池的方式，但是出现以下两个问题
```
CPU 资源大量浪费在阻塞等待上，导致 CPU 资源利用率低。在Java8之前，一般会通过回调的方式来减少阻塞，但是大量使用回调，又引发臭名昭著的回调地狱问题，导致代码可读性和可维护性大大降低。

为了增加并发度，会引入更多额外的线程池，随着 CPU 调度线程数的增加，会导致更严重的资源争用，宝贵的 CPU 资源被损耗在上下文切换上，而且线程本身也会占用系统资源，且不能无限增加。
```
所以当我们使用线程池的时候会导致CPU的上下文切换，那么CompletableFuture是如何解决上下文切换的问题呢？

文章结尾的时候说的是服务异步化之后一方面降低了响应时间，另一方面说的是降低了机器数量，其实我对这个还是很感兴趣的。为什么可以减少机器呢？减少机器这个角度是不是说通过异步化之后，之前的QPS可以通过异步提升，所以就不需要那么机器来响应相同的请求。这里的逻辑就是如果我之前需要调用30个接口，每个接口30ms，如果我只有仪态机器，那么在900ms只能响应一个请求，如果我想在1s内响应两个请求，那么我就需要增加一台机器。如果我现在优化快了一倍，那么我就能在一台机器在450ms内响应一个请求，在1s内就可以响应两个请求。因此需要的机器相应的减少一倍。

其实这里一直提到CompletableFuture的异步，那么在底层是如何实现异步的呢？比方说在CPU层面是如何实现异步的呢？其实我们还需要从操作系统层面去理解这个异步的概念，否则这个事情就无法很好的理解。CPU的调用过程是如果这个程序没有执行完成，那么会一直执行。

可以看一下《操作系统概念》，因为操作系统都是采用时分复用的，也就是说将时间分成多个大小相同的段，然后每个程序占用其中的一段，如果一个程序在指定的时间片段没有执行完成，CPU就需要将这个进程的上下文换成另一个进程的上下文。如果这里多个线程都因为远程调用占用的时间比较长，那么CPU就是在空转。这里所说的空转并不是说程序啥也不干，而是说CPU在不断地切换上下文，然后查看网络IO是否完成，如果完成，则当前线程就可以回收，如果没有完成，就需要在下一次继续调度这个线程。

其实我们说的程序会一直阻塞是因为这台机器除了运行这个程序（一个Main进程），然后就是运行业务中各个指定的线程池。比方说如果我们有4个接口需要去调用，那么这个时候如果是4个并行的接口，同时这四个借口都是没有依赖关系的，而只有一个主线程是依赖这四个接口的。其实这里使用多线程，但是每个线程都会阻塞，如果其中一个线程比较busy的话，其他业务就只能进入到等待队列了。但是如果我们使用的是异步的方式，那么就可以占用很少的线程，但是完成多个接口的请求，是这样吗？

## 源码解析
我们从单个依赖入手，了解CompletableFuture的原理
```java
    abstract static class Completion extends ForkJoinTask<Void>
        implements Runnable, AsynchronousCompletionTask {
        volatile Completion next;      // Treiber stack link

        /**
         * 会根据参数来决定是同步运行还是异步运行SYNC = 0，Async = 1，NESTED = 2
         * 全部交给子类去实现
         */
        abstract CompletableFuture<?> tryFire(int mode);

        /** Returns true if possibly still triggerable. Used by cleanStack. */
        abstract boolean isLive();
        /** 异步运行 */
        public final void run()                { tryFire(ASYNC); }
        /
        public final boolean exec()            { tryFire(ASYNC); return true; }
        public final Void getRawResult()       { return null; }
        public final void setRawResult(Void v) {}
    }
    // 这个抽线类继承了Completion
    abstract static class UniCompletion<T,V> extends Completion {
        // 这里为什么需要一个县城吃
        Executor executor;                 // executor to use (null if none)
        // 依赖的Future，也就是说如果这个dep完成了以后，就回执行
        CompletableFuture<V> dep;          // the dependent to complete
        // 那这个src的作用是什么呢？
        CompletableFuture<T> src;          // source for action

        UniCompletion(Executor executor, CompletableFuture<V> dep,
                      CompletableFuture<T> src) {
            this.executor = executor; this.dep = dep; this.src = src;
        }

        final boolean isLive() { return dep != null; }
    }
    static final class UniApply<T,V> extends UniCompletion<T,V> {
        Function<? super T,? extends V> fn;
        UniApply(Executor executor, CompletableFuture<V> dep,
                 CompletableFuture<T> src,
                 Function<? super T,? extends V> fn) {
            super(executor, dep, src); this.fn = fn;
        }
        final CompletableFuture<V> tryFire(int mode) {
            CompletableFuture<V> d; CompletableFuture<T> a;
            /*
             * 如果依赖的任务为null，或者调用CompletableFuture的uniApply方法为false
             */ 
            if ((d = dep) == null ||
                !d.uniApply(a = src, fn, mode > 0 ? null : this))
                return null;
            dep = null; src = null; fn = null;
            return d.postFire(a, mode);
        }
    }
```

```java
    final <S> boolean uniApply(CompletableFuture<S> a,
                               Function<? super S,? extends T> f,
                               UniApply<S,T> c) {
        Object r; Throwable x;
        if (a == null || (r = a.result) == null || f == null)
            return false;
        tryComplete: if (result == null) {
            if (r instanceof AltResult) {
                if ((x = ((AltResult)r).ex) != null) {
                    completeThrowable(x, r);
                    break tryComplete;
                }
                r = null;
            }
            try {
                if (c != null && !c.claim())
                    return false;
                @SuppressWarnings("unchecked") S s = (S) r;
                completeValue(f.apply(s));
            } catch (Throwable ex) {
                completeThrowable(ex);
            }
        }
        return true;
    }
```

```java
    final CompletableFuture<T> postFire(CompletableFuture<?> a, int mode) {
        if (a != null && a.stack != null) {
            if (mode < 0 || a.result == null)
                a.cleanStack();
            else
                a.postComplete();
        }
        if (result != null && stack != null) {
            if (mode < 0)
                return this;
            else
                postComplete();
        }
        return null;
    }
    final void cleanStack() {
        for (Completion p = null, q = stack; q != null;) {
            Completion s = q.next;
            if (q.isLive()) {
                p = q;
                q = s;
            }
            else if (p == null) {
                casStack(q, s);
                q = stack;
            }
            else {
                p.next = s;
                if (p.isLive())
                    q = s;
                else {
                    p = null;  // restart
                    q = stack;
                }
            }
        }
    }
    final void postComplete() {
        /*
         * On each step, variable f holds current dependents to pop
         * and run.  It is extended along only one path at a time,
         * pushing others to avoid unbounded recursion.
         */
        CompletableFuture<?> f = this; Completion h;
        while ((h = f.stack) != null ||
               (f != this && (h = (f = this).stack) != null)) {
            CompletableFuture<?> d; Completion t;
            if (f.casStack(h, t = h.next)) {
                if (t != null) {
                    if (f != this) {
                        pushStack(h);
                        continue;
                    }
                    h.next = null;    // detach
                }
                f = (d = h.tryFire(NESTED)) == null ? this : d;
            }
        }
    }
```

## 原理

### supplyAsync
昨天一直在用CompletableFuture，但是对于其中的方法supplyAsync、thenApply等方法的使用有着一些疑问。如果只是但是的调用supplyAsync方法，那么里面的Supplier会不会马上执行呢。我们可以先看一下下面的方法
```java
    public void test2() {
        ThreadPoolExecutor threadPoolExecutor = ThreadPoolManager.getSynchronousExecutor();
        CompletableFuture<String> futureA = CompletableFuture.supplyAsync(() -> {
            log.info("start execute futureA");
            sleep(100);
            // 返回结果
            return "result";
        }, threadPoolExecutor);
        // 主线程睡眠300ms，防止main线程退出
        sleep(300);
    }
```
这个方法会在console输出"start execute futureA"，所以我们来看看这个CompletableFuture在调用supply方法的整个流程。
```java
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                       Executor executor) {
        return asyncSupplyStage(screenExecutor(executor), supplier);
    }
    /**
     * 如果使用的是commonPool，那么使用的是ForkJoinPool，如果指定了ThreadPoolExecutor
     * 如果明确指定了ThreadPoolExecutor，那么需要对线程池判空。
     */
    static Executor screenExecutor(Executor e) {
        if (!useCommonPool && e == ForkJoinPool.commonPool())
            return asyncPool;
        if (e == null) throw new NullPointerException();
        return e;
    }
    /**
     * 这里先创建了一个CompletableFuture，然后将任务提交给线程池
     */
    static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                     Supplier<U> f) {
        if (f == null) throw new NullPointerException();
        CompletableFuture<U> d = new CompletableFuture<U>();
        // 其实这里就是将supplier封装了一层，同时创建一个CompletableFuture
        e.execute(new AsyncSupply<U>(d, f));
        return d;
    }
    /**
     * 
     */
    static final class AsyncSupply<T> extends ForkJoinTask<Void>
            implements Runnable, AsynchronousCompletionTask {
        CompletableFuture<T> dep; Supplier<T> fn;
        AsyncSupply(CompletableFuture<T> dep, Supplier<T> fn) {
            this.dep = dep; this.fn = fn;
        }

        public final Void getRawResult() { return null; }
        public final void setRawResult(Void v) {}
        public final boolean exec() { run(); return true; }

        public void run() {
            CompletableFuture<T> d; Supplier<T> f;
            if ((d = dep) != null && (f = fn) != null) {
                dep = null; fn = null;
                if (d.result == null) {
                    try {
                        d.completeValue(f.get());
                    } catch (Throwable ex) {
                        d.completeThrowable(ex);
                    }
                }
                d.postComplete();
            }
        }
    }
```

* 首先判断是否使用ForkJoinPool，如果没有，则查看指定的线程池是否为null。
* 然后创建一个CompletableFuture对象，同时将Supplier方程式接口和CompletableFuture实现类构建一个AsyncSupply对象。
* 将AsyncSupply对象传递给ThreadPoolExecutore执行，那么上面的CompletableFuture就是用来记录上面。

所以在看完supplyAsync方法时你会发现，其实这种方式和之前Thread执行方式没有什么不同，Thread调用的是自己的方法执行Runnable，CompletableFuture执行的是
### runAsync
runAsync和前面的supplyAsync之间的区别是runAsync没有返回结果，而supplyAyns会有返回结果。同样提供了指定了线程池还是使用默认线程池两种方式。
```java

    public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return asyncRunStage(asyncPool, runnable);
    }

    static CompletableFuture<Void> asyncRunStage(Executor e, Runnable f) {
        if (f == null) throw new NullPointerException();
        CompletableFuture<Void> d = new CompletableFuture<Void>();
        e.execute(new AsyncRun(d, f));
        return d;
    }
    static final class AsyncRun extends ForkJoinTask<Void>
            implements Runnable, AsynchronousCompletionTask {
        CompletableFuture<Void> dep; Runnable fn;
        AsyncRun(CompletableFuture<Void> dep, Runnable fn) {
            this.dep = dep; this.fn = fn;
        }

        public final Void getRawResult() { return null; }
        public final void setRawResult(Void v) {}
        public final boolean exec() { run(); return true; }

        public void run() {
            CompletableFuture<Void> d; Runnable f;
            if ((d = dep) != null && (f = fn) != null) {
                dep = null; fn = null;
                if (d.result == null) {
                    try {
                        f.run();
                        d.completeNull();
                    } catch (Throwable ex) {
                        d.completeThrowable(ex);
                    }
                }
                d.postComplete();
            }
        }
    }    
```

### thenAccept
thenAccept方法接收一个Consumer方程式接口实现类。
```java
    public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
        return uniAcceptStage(null, action);
    }
    private CompletableFuture<Void> uniAcceptStage(Executor e,
                                                   Consumer<? super T> f) {
        if (f == null) throw new NullPointerException();
        // 创建的这个CompletableFuture主要用于监听Consumer的执行情况
        CompletableFuture<Void> d = new CompletableFuture<Void>();
        if (e != null || !d.uniAccept(this, f, null)) {
            UniAccept<T> c = new UniAccept<T>(e, d, this, f);
            // 将Consumer包装成一个UniAccept类，然后添加到当前的栈中
            push(c);
            // 同步执行
            c.tryFire(SYNC);
        }
        return d;
    }
```

### thenApply
### thenCombine
### thenRun

```java

```


## 性能测试对比
### 性能测试1
性能测试1分别使用CompletableFuture、Callable执行多线程并发。同时控制两个变量，一个是针对任务数量，另一个就是针对线程睡眠时长来判断。
CompletableFuture测试类
```java
    public void testCompletableFuture1() {
        ThreadPoolExecutor threadPoolExecutor = ThreadPoolManager.getSynchronousExecutor();
        long start = System.currentTimeMillis();
        List<CompletableFuture<Void>> futures = createFuture(1000, 500, threadPoolExecutor);
        CompletableFuture<Object> result = CompletableFuture.allOf(futures.toArray(new CompletableFuture<?>[0]))
                .thenApply(v -> futures.stream()
                        .map(CompletableFuture::join)
                        .collect(Collectors.toList())
                );
        result.join();
        log.info("total time::{}", System.currentTimeMillis() - start);
        System.exit(0);
    }

    public void testCompletableFuture2() {
        ThreadPoolExecutor threadPoolExecutor = ThreadPoolManager.getSynchronousExecutor();
        long start = System.currentTimeMillis();
        List<CompletableFuture<Void>> futures = createFuture(1000, 100, threadPoolExecutor);
        CompletableFuture<Object> result = CompletableFuture.allOf(futures.toArray(new CompletableFuture<?>[0]))
                .thenApply(v -> futures.stream()
                        .map(CompletableFuture::join)
                        .collect(Collectors.toList())
                );
        result.join();
        log.info("total time::{}", System.currentTimeMillis() - start);
        System.exit(0);
    }

    List<CompletableFuture<Void>> createFuture(int round, int sleep, ThreadPoolExecutor threadPoolExecutor) {
        List<CompletableFuture<Void>> futures = new ArrayList<>(round);
        for (int i = 0; i < round; i++) {
            CompletableFuture<Void> futureA = CompletableFuture.supplyAsync(() -> {
                try {
                    // 模拟耗时操作
                    Thread.sleep(sleep);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 返回结果
                return null;
            }, threadPoolExecutor);
            futures.add(futureA);
        }
        return futures;
    }
```
|方法|轮询次数|睡眠时长（单位：ms）|运行时长｜
|  ----  | ----  | ----  | ----  |
| CompletableFuture|1000| 500| 20681|
| Callable | 1000 | 500 | 20647|
| CompletableFuture|1000| 100| 4552|
| Callable | 1000 | 100 | 4458|    


从这个上面我们可以看到基本没有什么区别的。甚至使用Callable的方式还要更加好一些。所以如果只是单纯的通过线程池调用的方式，那么这种方式是没有什么意义的。
### 性能测试2
我们外层使用一个业务线程池，这个线程池主要处理业务逻辑。模仿线上情况的Dubbo调用的异步方式，我们使用一个核心线程数为300的线程池，这样就能够模仿内部调用情况

Completable测试方法
```java
    public void testCompletableFuture3() {
        ThreadPoolExecutor businessThreadPool = ThreadPoolManager.getSynchronousExecutor();
        ThreadPoolExecutor rpcThreadPool = new ThreadPoolExecutor(300,
                600,
                60,
                TimeUnit.SECONDS, new SynchronousQueue<>(),
                new ThreadPoolExecutor.CallerRunsPolicy());
        long start = System.currentTimeMillis();
        asyncInvoke(1000, 500, rpcThreadPool, businessThreadPool);
        log.info("total time::{}", System.currentTimeMillis() - start);
    }

    public void testCompletableFuture4() {
        ThreadPoolExecutor businessThreadPool = ThreadPoolManager.getSynchronousExecutor();
        ThreadPoolExecutor rpcThreadPool = new ThreadPoolExecutor(300,
                600,
                60,
                TimeUnit.SECONDS, new SynchronousQueue<>(),
                new ThreadPoolExecutor.CallerRunsPolicy());
        long start = System.currentTimeMillis();
        asyncInvoke(1000, 100, rpcThreadPool, businessThreadPool);
        log.info("total time::{}", System.currentTimeMillis() - start);
    }

    private void asyncInvoke(int round, int sleepTime, ThreadPoolExecutor rpcThreadPool, ThreadPoolExecutor businessThreadPool) {
        List<Callable<Void>> callables = new ArrayList<>(round);
        for (int i = 0; i < round; i++) {
            Callable<Void> callable = () -> {
                List<CompletableFuture<Void>> rpcFutures = new ArrayList<>();
                CompletableFuture<Void> futureA = CompletableFuture.supplyAsync(() -> {
                    try {
                        // 模拟耗时操作
                        Thread.sleep(sleepTime);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 返回结果
                    return null;
                }, rpcThreadPool);
                rpcFutures.add(futureA);
                CompletableFuture<Void> futureB = CompletableFuture.supplyAsync(() -> {
                    try {
                        // 模拟耗时操作
                        Thread.sleep(sleepTime);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 返回结果
                    return null;
                }, rpcThreadPool);
                rpcFutures.add(futureB);
                CompletableFuture<Void> futureC = CompletableFuture.supplyAsync(() -> {
                    try {
                        // 模拟耗时操作
                        Thread.sleep(sleepTime);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 返回结果
                    return null;
                }, rpcThreadPool);
                rpcFutures.add(futureC);
                CompletableFuture<List<Void>> rpcFuture = sequence(rpcFutures);
                rpcFuture.get();
                return null;
            };
            businessThreadPool.submit(callable);
        }
    }
```
Callable测试方法
```java
public void testSyncCallable3() {
        ThreadPoolExecutor businessThreadPool = ThreadPoolManager.getSynchronousExecutor();
        ThreadPoolExecutor rpcThreadPool = new ThreadPoolExecutor(300,
                600,
                60,
                TimeUnit.SECONDS, new SynchronousQueue<>(),
                new ThreadPoolExecutor.CallerRunsPolicy());
        long start = System.currentTimeMillis();
        asyncInvoke(1000, 500, rpcThreadPool, businessThreadPool);
        log.info("total time::{}", System.currentTimeMillis() - start);
    }

    public void testSyncCallable4() {
        ThreadPoolExecutor businessThreadPool = ThreadPoolManager.getSynchronousExecutor();
        ThreadPoolExecutor rpcThreadPool = new ThreadPoolExecutor(300,
                600,
                60,
                TimeUnit.SECONDS, new SynchronousQueue<>(),
                new ThreadPoolExecutor.CallerRunsPolicy());
        long start = System.currentTimeMillis();
        asyncInvoke(1000, 100, rpcThreadPool, businessThreadPool);
        log.info("total time::{}", System.currentTimeMillis() - start);
    }

    private void syncInvoke(int round, int sleepTime, ThreadPoolExecutor rpcThreadPool, ThreadPoolExecutor businessThreadPool) {
        List<Callable<Void>> callables = new ArrayList<>(round);
        for (int i = 0; i < round; i++) {
            Callable<Void> callable = () -> {
                List<Callable<Void>> rpcCallables = new ArrayList<>();
                Callable<Void> callableA = () -> {
                    try {
                        // 模拟耗时操作
                        Thread.sleep(sleepTime);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 返回结果
                    return null;
                };
                rpcCallables.add(callableA);

                Callable<Void> callableB = () -> {
                    try {
                        // 模拟耗时操作
                        Thread.sleep(sleepTime);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 返回结果
                    return null;
                };
                rpcCallables.add(callableB);
                Callable<Void> callableC = () -> {
                    try {
                        // 模拟耗时操作
                        Thread.sleep(sleepTime);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 返回结果
                    return null;
                };
                rpcCallables.add(callableC);
                List<Future<Void>> futures = rpcThreadPool.invokeAll(rpcCallables);
                futures.stream().map(future -> {
                    try {
                        future.get();
                    } catch (Exception e) {
                        log.error("invoke future method error", e);
                    }
                    return null;
                }).collect(Collectors.toList());
                return null;
            };
            businessThreadPool.submit(callable);
        }
    }
```

|方法|轮询次数|睡眠时长（单位：ms）|运行时长｜
|  ----  | ----  | ----  | ----  |
| CompletableFuture|1000| 500| 20253|
| Callable | 1000 | 500 | 22212|
| CompletableFuture|1000| 200| 8803|  
| Callable | 1000 | 200 | 8994|  
| CompletableFuture|1000| 100| 4642|
| Callable | 1000 | 100 | 4636|    
