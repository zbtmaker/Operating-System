# 一、ReentrantLock源码解读

## 非公平锁获取锁流程
```java
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }       
```
从上面的代码可以看出，程序首先通过compareAndSet方式将当前线程设置成一个独占线程，其中compareAndSetState方法是调用了Unsafe类实现的。如果通过compareAndSetState方法设置成功，那么将当前的线程设置成独占线程。

如果设置失败，则会调用acquire方法来尝试重新设置独占线程或者是设置失败则将当前线程添加到等待队列当中。
```java
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```
这里首先会调用tryAcquire方法，这个方法会再一次尝试使用compareAndSetState方法来设置线程独占锁，源码如下：
```java
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 尝试设置state参数，设置成功则当前线程独占锁
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果当前线程已经独占锁，那么增加当前线程独占锁的次数（此方法主要是递归调用出现）
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
```
* 如果当前锁没有被任何线程占有，首先还是调用compareAndSetState方法设置线程独占锁；
* 如果当前锁已经被线程占有，增加当前线程占有锁的次数，因为一个线程重复占有锁（重入锁）；
* 如果上面都失败了，那么说明此次线程独占锁的操作失败了，此时返回false。

上面如果独占锁失败，会调用acquireQueued方法将当前线程添加到等待队列中。我们看一下源码分析
```java
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            // 通过Unsafe的compareAndSet方法设置队尾节点
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 如果对队尾节点为空，证明此时节点应该
        enq(node);
        return node;
    }

    private Node enq(final Node node) {
        for (;;) {
            // 如果队头和队尾都为空，那么将当前节点设置成头节点
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                // 将当前节点的节点设置成队尾节点，然后将当前节点尝试设置成队尾节点
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```
上面的acquireQueued(addWaiter(Node.EXCLUSIVE), arg)方法首先会调用addWaiter方法将当前线层入队；
* 如果尾节点不为空，说明前面还有线程在等待获取锁，因此尝试使用Unsafe的compareAndSet方法将当前节点设置为尾节点，这里只是进行一次尝试，如果尝试成功，那么就直接可以将当前节点返回；
* 如果设置不成功，那么就需要进入到循环里面，通过反复的尝试将节点添加到队列当中，这里其实就是一个典型的使用轻量级锁入队的示范。每一次循环，如果判断尾节点为空，假设这时有两个节点都走到了t == null的逻辑，两个节点只有一个节点执行compareAndSetHead成功，因此这个时候头节点被更新了，同时尾节点也被更新了，然后线程。那么有没有可能，一个线程执行到compareAndSetHead这个方法成功，另一个节点执行compareAndSetHead方法失败，同时，另一个节点因为循环又走到了t=null这个指令了，然后又判断一次，从而又执行一次compareAndSetHead方法。相当于创建了多个为空的Node（这个Node的线程引用为空）。
* 我们假设上面的情况不会出现，现在两个线程再一次进入for循环，因为在上一轮竞争过程中已经创建了一个空的Node（头节点和尾节点都执行这个空节点），因此就会执行下面将节点添加到添加到队列的尾部。队列的尾部。

入队完成之后如果当前节点已经是首节点，那么此时再一次尝试获取锁，如果获取成功，那么首先需要将Header节点设置为null，
```java
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                // 如果获取锁的节点是头节点，获取独占锁成功则设置当前当前head节点的下一个节点，同时返回false
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 判断前一个节点的状态，然后决定是否要暂停当前线程，同时判断当前线程是否要中断。
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
这里我们可以看到，当一个节点进入队列之后，还是会不断的运行for循环，在这个循环过程中：
* 如果当前线程的头节点是Head，因为等待的线程在入队列时会进入队列时会创建一个空的头节点，如果说node的头节点是一个Head节点，那么表示在等待队列的线程只有当前线程了，因此需要奖当前节点设置成头节点，同时返回false
* 如果上面失败了，那么需要调用shouldParkAfterFailedAcquire方法判断这个节点是否需要调用park方法来暂停这个线程的执行。


我们来看一下shouldParkAfterFailedAcquire方法和parkAndCheckInterrupt方法做了什么。
```java
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        // 如果线程所在节点的前一个节点的处于single状态，那么表明当前线程的需要被暂停，
        // 这样就不需要在上面acquireQueue方法中一直进入for循环，如果被暂停了，此时CPU
        // 应该就不需要分配时间片给当前线程来无限的空转。
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        // 如果当前线程所在节点的状态处于取消状态，那么就需要将节点node与pred的
        // predecessor节点进行关联，直到这个节点node关联的前驱节点不再是取消状态
        // 这里有意思的是为什么这个节点在关联前驱节点为什么没有判断前驱节点是否是null呢
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            // 如果pred节点的状态不是SIGNAL、CANCEL状态，尝试将前一个节点设置成SIGNAL状态？
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

* 这里首先判断pred节点的状态是否是SIGNAL，如果前一个节点是SIGNAL状态，那么node节点也应就应该进入暂停状态；
* 如果node的前驱节点pred的状态>0，也就是说CANCEL状态，此时就需要将node节点关联到前面状态不为CANCEL的前驱节点；
* 如果pred节点的状态既不是SIGNAL，也不是CANCEL状态，那么就需要将pred节点的状态设置成SIGNAL状态？

因为pred节点是SIGNAL状态，所以这个节点就需要调用parkAndCheckInterrupt方法来暂停线程。
```java
    private final boolean parkAndCheckInterrupt() {
        // park线程，其实就是暂停当前线程
        LockSupport.park(this);
        // 判断当前线程是否已经被中断了
        return Thread.interrupted();
    }
```

## 非公平锁释放流程


```java
    public void unlock() {
        sync.release(1);
    }
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```
我们首先看一下tryRelease方法
```java

    protected final boolean tryRelease(int releases) {
        // 首先把获取锁的次数-1
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        // 如果获取锁的次数已经为0了，表示这个线程已经完全释放线程了，此时将线程独占标识符去掉
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        // 设置状态，这里为什么最后才设置状态呢？是因为防止其他线程在获取锁的时候提前获取到state值，从而出现不一致性
        setState(c);
        return free;
    }
```
* 首先把持有线程的次数state；
* 然后判断次有线程的数量c是否为零，如果为零，则说明当前线程已经完全释放完线程了，将独占线程的标识设置为null，这样就能够让其他线程在竞争锁时设置为独占线程；
* 最后将state回写到内存中，这样就能够让其他线程在竞争线程成功时确保上一个线程已经完全释放线程了。

我们接下来看一下unparkSuccessor方法
```java
    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```


