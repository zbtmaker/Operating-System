# CompletableFuture阅读笔记（三）-thenApply操作

## 

```java
    /**
     * 同步方法，不指定线程池，使用ForkJoinPool，传入consumer函数式接口实现
     */
    public <U> CompletableFuture<U> thenApply(
        Function<? super T,? extends U> fn) {
        return uniApplyStage(null, fn);
    }

    /**
     * 异步方法，不指定线程池，使用ForkJoinPool，传入Function函数式接口实现
     */
    public <U> CompletableFuture<U> thenApplyAsync(
        Function<? super T,? extends U> fn) {
        return uniApplyStage(asyncPool, fn);
    }

    /**
     * 异步方法，指定线程池，使用指定线程池执行，传入Function函数式接口实现
     */
    public <U> CompletableFuture<U> thenApplyAsync(
        Function<? super T,? extends U> fn, Executor executor) {
        return uniApplyStage(screenExecutor(executor), fn);
    }
```

## uniApplyStage

```java
    private <V> CompletableFuture<V> uniApplyStage(
        Executor e, Function<? super T,? extends V> f) {
        if (f == null) throw new NullPointerException();
        CompletableFuture<V> d =  new CompletableFuture<V>();
        if (e != null || !d.uniApply(this, f, null)) {
            UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
            push(c);
            c.tryFire(SYNC);
        }
        return d;
    }

    @SuppressWarnings("serial")
    static final class UniApply<T,V> extends UniCompletion<T,V> {
        Function<? super T,? extends V> fn;
        UniApply(Executor executor, CompletableFuture<V> dep,
                 CompletableFuture<T> src,
                 Function<? super T,? extends V> fn) {
            super(executor, dep, src); this.fn = fn;
        }
        final CompletableFuture<V> tryFire(int mode) {
            CompletableFuture<V> d; CompletableFuture<T> a;
            if ((d = dep) == null ||
                !d.uniApply(a = src, fn, mode > 0 ? null : this))
                return null;
            dep = null; src = null; fn = null;
            return d.postFire(a, mode);
        }
    }
```

## uniApply

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

我们看一下如果当前Function依赖的CompletableFuture报异常之后会出现什么情况
```java

    public void testException(){
        ThreadPoolExecutor threadPoolExecutor = ThreadPoolManager.getSynchronousExecutor();
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(new Supplier<String>() {
            @SneakyThrows
            @Override
            public String get() {
                throw new Exception("future1's exception");
            }
        }, threadPoolExecutor);

        CompletableFuture<String> future2 = future1.thenApplyAsync(t -> t, threadPoolExecutor);
        try{
            String res2 = future2.get();
        }catch(Exception ex) {
            log.error("执行future2的get方法异常",ex.getCause());
        }

        System.exit(0);
    }
```
在这个案例中，future1会抛出异常，然后执行获取future2的get方法时会抛出异常。