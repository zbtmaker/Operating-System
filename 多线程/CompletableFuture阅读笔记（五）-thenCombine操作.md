# CompletableFuture阅读笔记（五）-thenRun操作

## thenCombine
thenCombine方法主要接受一个BiFuction和一个CompletionStage，BiFunction是待执行的function，依赖的是调用thenCombine方法的CompletableFuture和传递进来的CompletionStage接口。
```java
    public <U,V> CompletableFuture<V> thenCombine(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(null, other, fn);
    }

    public <U,V> CompletableFuture<V> thenCombineAsync(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(asyncPool, other, fn);
    }

    /**
     *  接受一个CompletionStage、一个BiFunction、指定线程池
     */
    public <U,V> CompletableFuture<V> thenCombineAsync(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn, Executor executor) {
        return biApplyStage(screenExecutor(executor), other, fn);
    }
```

## biApplyStage

```java
    private <U,V> CompletableFuture<V> biApplyStage(
        Executor e, CompletionStage<U> o,
        BiFunction<? super T,? super U,? extends V> f) {
        CompletableFuture<U> b;
        if (f == null || (b = o.toCompletableFuture()) == null)
            throw new NullPointerException();
        CompletableFuture<V> d = new CompletableFuture<V>();
        if (e != null || !d.biApply(this, b, f, null)) {
            BiApply<T,U,V> c = new BiApply<T,U,V>(e, d, this, b, f);
            /**
             * 这个参数b就是CompletionStage获取CompletableFuture的结果
             * 这里将封装了BiFunction的BiApply对象和另一个CompletableFuture执行
             * bipush操作
             */ 
            bipush(b, c);
            c.tryFire(SYNC);
        }
        return d;
    }
```
* biApplyStage方法和之前的uniAcceptStage、uniApplyStage、uniRunStage一致，都是先判断依赖的CompletableFuture是否已经执行完成，如果执行完成，则将依赖的CompletableFuture的结果作为函数式接口（Function、Comsumer、Runnable）的参数传递给函数式接口。
* 如果依赖的CompletableFuture未执行完成，则将创建的（UniAccept、UniApply、UniRun、BiApply）压入到未执行完成的CompletableFuture的栈中。
* 最后再尝试执行tryFire方法

## BiApply
```java
    @SuppressWarnings("serial")
    static final class BiApply<T,U,V> extends BiCompletion<T,U,V> {
        BiFunction<? super T,? super U,? extends V> fn;
        BiApply(Executor executor, CompletableFuture<V> dep,
                CompletableFuture<T> src, CompletableFuture<U> snd,
                BiFunction<? super T,? super U,? extends V> fn) {
            super(executor, dep, src, snd); this.fn = fn;
        }
        final CompletableFuture<V> tryFire(int mode) {
            // d是BiApply用来承接BiFunction的CompletableFuture
            CompletableFuture<V> d;
            // a是调用biApplyStage生成BiApply的CompletableFuture
            CompletableFuture<T> a;
            // b是BiApply依赖的另一个CompletableFuture
            CompletableFuture<U> b;
            // 如果依赖的其中两个其中一个没有执行完成，
            if ((d = dep) == null ||
                !d.biApply(a = src, b = snd, fn, mode > 0 ? null : this))
                return null;
            dep = null; src = null; snd = null; fn = null;
            return d.postFire(a, b, mode);
        }
    }
    

    final <R,S> boolean biApply(CompletableFuture<R> a,
                                CompletableFuture<S> b,
                                BiFunction<? super R,? super S,? extends T> f,
                                BiApply<R,S,T> c) {
        Object r, s; Throwable x;
        if (a == null || (r = a.result) == null ||
            b == null || (s = b.result) == null || f == null)
            return false;
        tryComplete: if (result == null) {
            if (r instanceof AltResult) {
                // 如果异常执行了
                if ((x = ((AltResult)r).ex) != null) {
                    // 其实这里是将dep这个CompletableFuture的result设置成Exception
                    completeThrowable(x, r);
                    break tryComplete;
                }
                r = null;
            }
            if (s instanceof AltResult) {
                if ((x = ((AltResult)s).ex) != null) {
                    // 其实这里是将dep这个CompletableFuture的result设置成Exception
                    completeThrowable(x, s);
                    break tryComplete;
                }
                s = null;
            }
            try {
                if (c != null && !c.claim())
                    return false;
                @SuppressWarnings("unchecked") R rr = (R) r;
                @SuppressWarnings("unchecked") S ss = (S) s;
                completeValue(f.apply(rr, ss));
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
```
不管是双元依赖还是一元依赖，一旦依赖的CompletableFuture报异常以后，当前的CompletableFuture在执行的时候就会报异常。

## bipush
```java
    final void bipush(CompletableFuture<?> b, BiCompletion<?,?,?> c) {
        if (c != null) {
            Object r;
            /**
             * 如果当前的Completion没有执行完成，或者是将BiCompletion入栈，
             * 如果执行失败，那么就会一直执行入栈操作。
             */
            while ((r = result) == null && !tryPushStack(c))
                lazySetNext(c, null); // clear on failure
            /**
             * 1、先将BiCompletion添加到另一个CompletableFuture的栈中。
             * 2、如果另一个CompletableFuture的没有执行完成，会创建一个CoCompletion然后执行操作，然后
             */
            if (b != null && b != this && b.result == null) {
                Completion q = (r != null) ? c : new CoCompletion(c);
                while (b.result == null && !b.tryPushStack(q))
                    lazySetNext(q, null); // clear on failure
            }
        }
    }
```