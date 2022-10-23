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

这里的逻辑还是比较简单，
* 如果当前节点的状态<0，那么将node的状态设置为0，也就是初始状态；
* 如果ws>=0，其实这里应该是从node节点找到第一个状态节点<0的节点；
* 找到满足条件的节点，然后将节点唤醒。


## Condition

### await-加入等待队列
```java
    public final void await() throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        // 将节点添加到等待节点队列中
        Node node = addConditionWaiter();
        // 因为将其余等待的节点释放
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        // 如果节点在同步队列当中，那么就不需要将当前线程执行park了
        while (!isOnSyncQueue(node)) {
            LockSupport.park(this);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        // 这个调用的是外部类的方法
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
    /**
     * 首先判断节点的状态是否是CONDITION，活着这个节点的前驱节点为null，那么表示这个节点是只存在等待队列中，此时将节点
     * 如果node.next不等于null，说明节点已经在同步队列当中，如果这个节点是同步队列的尾节点，那么就需要从同步队列的尾节点
     * 开始遍历，直到在同步队列中找到一个节点与node节点相等。还是很好奇，在什么条件下，一个节点即在同步队列中，又会在等待
     * 等待队列当中。
     */
    final boolean isOnSyncQueue(Node node) {
        // 如果没有其他线程执行signal方法，那么当前线程所在的节点的状态都是CONDITION状态，如果其他线程调用了signal方法，会将等待队列中的
        // 那么此时就会返回true。
        if (node.waitStatus == Node.CONDITION || node.prev == null)
            return false;
        if (node.next != null) // If has successor, it must be on queue
            return true;
        /*
         * node.prev can be non-null, but not yet on queue because
         * the CAS to place it on queue can fail. So we have to
         * traverse from tail to make sure it actually made it.  It
         * will always be near the tail in calls to this method, and
         * unless the CAS failed (which is unlikely), it will be
         * there, so we hardly ever traverse much.
         */
        return findNodeFromTail(node);
    }

    private int checkInterruptWhileWaiting(Node node) {
        return Thread.interrupted() ? (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :0;
    }

    final boolean transferAfterCancelledWait(Node node) {
        if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
            enq(node);
            return true;
        }
        /*
         * If we lost out to a signal(), then we can't proceed
         * until it finishes its enq().  Cancelling during an
         * incomplete transfer is both rare and transient, so just
         * spin.
         */
        while (!isOnSyncQueue(node))
            Thread.yield();
        return false;
    }

    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
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

* 这里首先将当前线程构建一个Node添加到等待队列中（注意这个等待队列和执行lock操作的同步队列不是同一个队列）。等待队列的节点的状态为CONDITION，而同步队列中节点状态为CANCEl、SIGNAL。等待队列为单向链表，使用nextWaiter连接，而同步队列中的节点通过prev和next两个指针连接是一个双向的链表。
* 因为当前线程调用的lock.Condition的wait方法，说明当前线程已经要放弃锁的占有，此时会调用fullyRelease方法将线程之前占有锁的次数全部清零，同时把线程独占的引用给释放。这里也有一个问题，如果这个线程因为重入的原因多次持有锁，那么当线程获取到锁以后，回到上一次执行的地方，那是不是在上一个方法栈需要重新获取锁。
* 然后判断当前线程是否在同步队列当中，前面我们知道等待队列和同步队列可以通过waitStatus和是否是单向链表来判断，因此如果一个这个节点的状态是CONDITION状态，那么这个节点肯定不在同步队列，或者这个节点的前驱节点prev==null说明这个节点处在单向链表中，那么这个节点就不在同步队列当中，反之，则在同步队列中。
* 因为这个节点已经添加到等待队列节点当中，那么这个节点就需要重新调用acquireQueue方法来重新进入同步队列当中。

### signal-移除等待队列


```java
    public final void signal() {
        // 如果当前线程没有获取锁就执行signal方法就会报异常
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            // 将首从等待队列中移除
            doSignal(first);
    }
```

```java
    // 更新首节点尾当前节点的下一个节点，同时唤醒当前节点
    private void doSignal(Node first) {
        do {
            // 更新等待队列的首节点，同时，如果first节点的下一个节点是null节点，那么说明first节点即是首节点，又是尾节点，因此将尾节点置空
            if ( (firstWaiter = first.nextWaiter) == null)
                lastWaiter = null;
            // 将first节点与等待队列断开
            first.nextWaiter = null;
        } while (!transferForSignal(first) && (first = firstWaiter) != null);
    }
```
执行唤醒操作
```java
    /**
     * 其实这个方法只有signal和signalAll两个方法使用，那么这个方法肯定是线程安全的，因为调用这个方法都需要获取锁的独占权
     */
    final boolean transferForSignal(Node node) {
        // 如果节点设置失败，那么表示这个节点已经被取消了，在调用signal活着signalAll这个操作之前已经执行了
        // CANCEL操作（中断或者执行超时），那么就返回false，然后让主流程唤醒下一个节点，将节点的状态设置为初始状态
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
            
        // 然后将之前在等待队列首节点的然后将节点添加到同步队列中，返回当前节点的前一个节点
        Node p = enq(node);
        int ws = p.waitStatus;
        // 如果节点p的状态既不是ws>0(CANCEL状态)，那么节点P的状态就可以被设置为SINGAL状态，如果设置失败，为什么会设置失败呢？因为这个时候，已经没有其
        // 他线程可以操作了，如果前一个节点是HEAD节点活着在做这个操作之前（前一个线程在执行的时候执行了中断或者超时）,但是如果这个节点是头节点的话，又该
        // 如何呢？
        // 
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
    /**
     *
     */
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```
首先我们看到，如果一个线程在获取锁之后执行了signal方法是没有放弃锁的。如果当前线程没有放弃锁，那么其他线程是不可能调用lock成功（也就是说其他线程会进入同步队列），同时也不会有线程调用了await方法（这个）

