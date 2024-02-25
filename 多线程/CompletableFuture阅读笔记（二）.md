CompletableFuture阅读笔记（二）-thenAccept操作

thenAccept操作有三种操作，同步和异步，异步方法中可以指定线程池，也可以使用默认线程池。
## 源码解析
我们从单个依赖入手，了解CompletableFuture的原理
```java
    /**
     * 同步执行，也就是当前CompletableFuture执行完成以后，则会同步执行Consumer中的action方法
     */
    public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
        return uniAcceptStage(null, action);
    }

    /**
     * 不指定线程池的情况下则使用当前CompletableFuture的默认线程池中的线程来执行Consumer的accept方法
     * 并不能保证当前CompletableFuture中的function和执行Consumer的accept方法是同一个线程。
     */
    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
        return uniAcceptStage(asyncPool, action);
    }

    /**
     * 指定线程池的情况下，会使用ThreadPool中的线程来执行当前的Consumer的accept方法
     */
    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,
                                                   Executor executor) {
        return uniAcceptStage(screenExecutor(executor), action);
    }
    /**
     * 1、首先会创建一个UniAccept类包装待执行的Consumer，
     * 
     */
    private CompletableFuture<Void> uniAcceptStage(Executor e,
                                                   Consumer<? super T> f) {
        if (f == null) throw new NullPointerException();
        CompletableFuture<Void> d = new CompletableFuture<Void>();
        if (e != null || !d.uniAccept(this, f, null)) {
            UniAccept<T> c = new UniAccept<T>(e, d, this, f);
            push(c);
            c.tryFire(SYNC);
        }
        return d;
    }
```
* 做了四件事情，创建一个CompletableFuture（d），主要用于执行Consumer之后返回，可以再次使用CompletableFuture的特性
* 创建一个UniAccept对象，这个对象包括了当前CompletableFuture（this）、Consumer（f）、执行Consumer（f）
的线程池，以及上一步创建的CompletableFuture（d）
* 将上一步创建好的UniAccept对象（c）,压到当前CompletableFuture（c）的栈中，待CompletableFuture执行完成以后会将栈中的对象一一弹出，然后执行。
* 然后调用UniAccept对象的tryFire方法。

从上面的步骤我们看到，在将新创建的UniAccept压入当前CompletableFuture的栈之后，立马就执行了UniAccept对象的tryFire方法。我们从常识应该知道压入栈应该不会做while（true）循环，而是直接压入当前CompletableFuture（c）的栈中。也就是说tryFire方法中要么采用了轮询的方式来查看当前CompletableFuture（c）是否已经执行完成。

### UniAccept

我们首先来看UniAccept类，这个类继承了UniCompletion
```java
    @SuppressWarnings("serial")
    static final class UniAccept<T> extends UniCompletion<T,Void> {
        Consumer<? super T> fn;
        UniAccept(Executor executor, CompletableFuture<Void> dep,
                  CompletableFuture<T> src, Consumer<? super T> fn) {
            // 调用父类的构造函数
            super(executor, dep, src); this.fn = fn;
        }
        final CompletableFuture<Void> tryFire(int mode) {
            CompletableFuture<Void> d; CompletableFuture<T> a;
            if ((d = dep) == null ||
                !d.uniAccept(a = src, fn, mode > 0 ? null : this))
                return null;
            dep = null; src = null; fn = null;
            return d.postFire(a, mode);
        }
    }

    abstract static class UniCompletion<T,V> extends Completion {
        Executor executor;                 // executor to use (null if none)
        CompletableFuture<V> dep;          // the dependent to complete
        CompletableFuture<T> src;          // source for action

        UniCompletion(Executor executor, CompletableFuture<V> dep,
                      CompletableFuture<T> src) {
            this.executor = executor; this.dep = dep; this.src = src;
        }

        /**
         * Returns true if action can be run. Call only when known to
         * be triggerable. Uses FJ tag bit to ensure that only one
         * thread claims ownership.  If async, starts as task -- a
         * later call to tryFire will run action.
         */
        final boolean claim() {
            Executor e = executor;
            if (compareAndSetForkJoinTaskTag((short)0, (short)1)) {
                if (e == null)
                    return true;
                executor = null; // disable
                e.execute(this);
            }
            return false;
        }

        final boolean isLive() { return dep != null; }
    }

    abstract static class Completion extends ForkJoinTask<Void>
        implements Runnable, AsynchronousCompletionTask {
        volatile Completion next;      // Treiber stack link

        /**
         * Performs completion action if triggered, returning a
         * dependent that may need propagation, if one exists.
         *
         * @param mode SYNC, ASYNC, or NESTED
         */
        abstract CompletableFuture<?> tryFire(int mode);

        /** Returns true if possibly still triggerable. Used by cleanStack. */
        abstract boolean isLive();

        public final void run()                { tryFire(ASYNC); }
        public final boolean exec()            { tryFire(ASYNC); return true; }
        public final Void getRawResult()       { return null; }
        public final void setRawResult(Void v) {}
    }
```

### push
CompletableFuture的push主要是首先要判断入参是否为空。如果result != null（表示当前CompletableFuture已经执行完成）那么创建的UniCompletion就不会入栈。如果执行tryPushStackl(c)执行失败，那么会一直使用CAS方式将UniCompletion压入栈。为了防止多线程并发写入问题，CompletableFuture使用了CAS而不是synchronized或者Lock来解决并发入栈问题。

```java
    final void push(UniCompletion<?,?> c) {
        if (c != null) {
            while (result == null && !tryPushStack(c))
                lazySetNext(c, null); // clear on failure
        }
    }
    final boolean tryPushStack(Completion c) {
        Completion h = stack;
        lazySetNext(c, h);
        return UNSAFE.compareAndSwapObject(this, STACK, h, c);
    }
    static void lazySetNext(Completion c, Completion next) {
        UNSAFE.putOrderedObject(c, NEXT, next);
    }
```
从这里可以看到push操作如果没有多线程，那么就不会一直执行while操作，那么监听的操作就是UniAccept类中的tryFire方法在监听CompletableFuture（c）的执行。



### postFire

```java
    static final class UniAccept<T> extends UniCompletion<T,Void> {
        // 省略构造函数、field，主要关注tryFire方法


        /**
         * 这里因该是为了防止UniAccept在加入到栈时依赖的CompletableFuture已经执行完毕
         * 如果依赖的CompletableFuture为null，那么就不执行下面的流程
         * 如果依赖的CompletableFuture还没有执行完成，此时tryFire方法返回null
         * 因为如果d.uniAccept执行以后返回false，那么说明UniAccept依赖的任务还没有执行完成
         * 因为如果d.uniAccept执行以后返回true，那么说明UniAccept依赖的任务执行完成，同时UniAccept也已经执行完成
         * 此时就应该执行postFire方法
         */
        final CompletableFuture<Void> tryFire(int mode) {
            CompletableFuture<Void> d; CompletableFuture<T> a;
            if ((d = dep) == null ||
                !d.uniAccept(a = src, fn, mode > 0 ? null : this))
                return null;
            dep = null; src = null; fn = null;
            return d.postFire(a, mode);
        }
    }
```
uniAccept方法

```java
    final <S> boolean uniAccept(CompletableFuture<S> a,
                                Consumer<? super S> f, UniAccept<S> c) {
        Object r; Throwable x;
        // 如果依赖的CompletableFuture还没有执行完成，那么返回false
        if (a == null || (r = a.result) == null || f == null)
            return false;
        
        tryComplete: if (result == null) {
            /** 
             * 如果Completable Future已经执行完成了，判断是否是正常执行
             * 如果异常执行，那么当前UniAccept的结果也要设置成Throwable
             */
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
                /** 
                 * 如果CompletableFuture执行完毕，则执行Consumer的Accept方法
                 * 同时CompletableFuture的结果需要作为Consumer的accept方法入参传递。
                 * 执行完成以后将UniAccept的结果设置成NIL（new AltResult(null)）
                 */
                @SuppressWarnings("unchecked") S s = (S) r;
                f.accept(s);
                completeNull();
            } catch (Throwable ex) {
                completeThrowable(ex);
            }
        }
        return true;
    }
    final boolean completeThrowable(Throwable x, Object r) {
        return UNSAFE.compareAndSwapObject(this, RESULT, null,
                                           encodeThrowable(x, r));
    }
    /**
     * 将CompletableFuture的Result设置成NIL
     */
    final boolean completeNull() {
        return UNSAFE.compareAndSwapObject(this, RESULT, null,
                                           NIL);
    }

    /** The encoding of the null value. */
    static final AltResult NIL = new AltResult(null);
```
postFire

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