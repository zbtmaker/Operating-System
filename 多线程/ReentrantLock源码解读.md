# 一、ReentrantLock源码解读

非公平锁获取锁流程
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
                // 
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