# 单机限流组件-RateLimiter

<!-- TOC -->

- [单机限流组件-RateLimiter](#单机限流组件-ratelimiter)
  - [StopWatch](#stopwatch)
  - [create](#create)
  - [acquire](#acquire)
  - [tryAcquire](#tryacquire)
  - [使用案例](#使用案例)
    - [tryAcquire使用用例](#tryacquire使用用例)
    - [acquire使用案例](#acquire使用案例)

<!-- /TOC -->

## StopWatch
StopWatch类主要的工作就是记录第$n$次访问，比方说第1次访问的时间是1673668800ms（这里使用的是服务器的时间戳），第3次访问的时间是1673669100ms，通过这个类提供的方法就可以获取时间差300ms。

RateLimiter有一个直接实现类，拥有一个SleepingStopWatch，这个类主要用于做什么呢？这个类主要对StopWatch进行了包装，主要是增加了sleepMicrosUninterruptibly这个方法。

```java
abstract static class SleepingStopwatch {
    /** Constructor for use by subclasses. */
    protected SleepingStopwatch() {}

    /*
     * 这个方法返回的是服务器当前时间距离创建RateLimiter时的时间
     */
    protected abstract long readMicros();

    protected abstract void sleepMicrosUninterruptibly(long micros);

    public static SleepingStopwatch createFromSystemTimer() {
      return new SleepingStopwatch() {
        // 这里StopWatch作为类的一个属性，而StopWatch主要的功能是计算两个时间差，相当于我们之前
        // 使用System.currentTimeMillis() - lastTime;
        final Stopwatch stopwatch = Stopwatch.createStarted();

        @Override
        protected long readMicros() {
          return stopwatch.elapsed(MICROSECONDS);
        }

        @Override
        protected void sleepMicrosUninterruptibly(long micros) {
          if (micros > 0) {
            Uninterruptibles.sleepUninterruptibly(micros, MICROSECONDS);
          }
        }
      };
    }
}
```
sleepUninterruptibly这个方法主要是提供一个不中断的线程休眠，其实这种方法有点好奇，线程被中断之后继续执行，然后继续执行Thread.sleep方法来实现线程休眠。
```java
public final class Uninterruptibles {
    public static void sleepUninterruptibly(long sleepFor, TimeUnit unit) {
    boolean interrupted = false;
    try {
      long remainingNanos = unit.toNanos(sleepFor);
      long end = System.nanoTime() + remainingNanos;
      while (true) {
        try {
          // TimeUnit.sleep() treats negative timeouts just like zero.
          NANOSECONDS.sleep(remainingNanos);
          return;
        } catch (InterruptedException e) {
          interrupted = true;
          remainingNanos = end - System.nanoTime();
        }
      }
    } finally {
      if (interrupted) {
        Thread.currentThread().interrupt();
      }
    }
  }
}
```
我们可以来看RateLimiter的创建方法
## create
我们在看这个方法前先把初始化的几个参数做一个说明
* storedPermits：两次请求能够存储的permit数量
* stableIntervalMicros：获取每个permit需要等到的时间，比方说设置的QPS=10 permit/s，也就是说1s能够获取10ms，那么stableIntervalMicros = 1000 / 10 = 100ms，也就是说生产一个permit需要耗费的时长为100ms。
* maxPermits：最多能够存储的permit的数量，如果我们设置的QPS = 10 permit/s，那么maxPermit = 10。
```java
public class RateLimiter {
  public static RateLimiter create(double permitsPerSecond) {
    return create(permitsPerSecond, SleepingStopwatch.createFromSystemTimer());
  }

  @VisibleForTesting
  static RateLimiter create(double permitsPerSecond, SleepingStopwatch stopwatch) {
    // 这个方法创建的是SmoothBursty方式限流
    RateLimiter rateLimiter = new SmoothBursty(stopwatch, 1.0 /* maxBurstSeconds */);
    rateLimiter.setRate(permitsPerSecond);
    return rateLimiter;
  }

  public final void setRate(double permitsPerSecond) {
    checkArgument(
        permitsPerSecond > 0.0 && !Double.isNaN(permitsPerSecond), "rate must be positive");
    synchronized (mutex()) {
      doSetRate(permitsPerSecond, stopwatch.readMicros());
    }
  }
}
```
我们可以看到，在创建RateLimiter时，首先需要创建一个SleepingStopWatch对象作为其属性，然后创建一个RateLimiter的子类，我们可以看一下这个子类的具体实现

```java
abstract class SmoothRateLimiter extends RateLimiter {

  /** 在创建RateLimiter时设置的QPS，如果设置的QPS=5，那么此属性的值就是5. */
  double storedPermits;

  /** The maximum number of stored permits. */
  double maxPermits;

  /**
   * 如果我们一秒允许获取10个permits，那么获取一个permit的间隔就是100ms。
   */
  double stableIntervalMicros;

  
  static final class SmoothBursty extends SmoothRateLimiter {
    /** The work (permits) of how many seconds can be saved up if this RateLimiter is unused? */
    final double maxBurstSeconds;

    SmoothBursty(SleepingStopwatch stopwatch, double maxBurstSeconds) {
      super(stopwatch);
      this.maxBurstSeconds = maxBurstSeconds;
    }

    @Override
    void doSetRate(double permitsPerSecond, double stableIntervalMicros) {
      double oldMaxPermits = this.maxPermits;
      maxPermits = maxBurstSeconds * permitsPerSecond;
      if (oldMaxPermits == Double.POSITIVE_INFINITY) {
        // if we don't special-case this, we would get storedPermits == NaN, below
        storedPermits = maxPermits;
      } else {
        storedPermits =
            (oldMaxPermits == 0.0)
                ? 0.0 // initial state
                : storedPermits * maxPermits / oldMaxPermits;
      }
    }
    @Override
    void doSetRate(double permitsPerSecond, double stableIntervalMicros) {
      double oldMaxPermits = this.maxPermits;
      maxPermits = maxBurstSeconds * permitsPerSecond;
      if (oldMaxPermits == Double.POSITIVE_INFINITY) {
        // if we don't special-case this, we would get storedPermits == NaN, below
        storedPermits = maxPermits;
      } else {
        storedPermits =
            (oldMaxPermits == 0.0)
                ? 0.0 // initial state
                : storedPermits * maxPermits / oldMaxPermits;
      }
    }
  }

  @Override
  final void doSetRate(double permitsPerSecond, long nowMicros) {
    resync(nowMicros);
    double stableIntervalMicros = SECONDS.toMicros(1L) / permitsPerSecond;
    this.stableIntervalMicros = stableIntervalMicros;
    doSetRate(permitsPerSecond, stableIntervalMicros);
  }

  void resync(long nowMicros) {
    // if nextFreeTicket is in the past, resync to now
    if (nowMicros > nextFreeTicketMicros) {
      // 两次访问时间间隔来计算冷却时间内积累了多少的permit。
      double newPermits = (nowMicros - nextFreeTicketMicros) / coolDownIntervalMicros();
      storedPermits = min(maxPermits, storedPermits + newPermits);
      nextFreeTicketMicros = nowMicros;
    }
  }
}
```
我们可以看一下上面的流程，
* 首先创建了一个SmoothBursty对象，这个对象是RateLimiter的一个子类，然后调用了RateLimiter的构造方法，创建了一个SleepingStopWatch
* 在创建SleepingStopWatch之后，如果程序多少秒没有调用RateLimiter，这个功能待定
* 然后调用了setRate方法设置QPS，RateLimiter的setRate是一个线程安全的方法，会调用RateLimiter的另一个方法doSetRate方法，这个方法在SmoothRateLimiter提供了实现
* 在在doSetRate方法中，首先调用了resync方法
  * 计算storedPermits，这个storedPermits计算方式就是看两次访问的时间间隔内积累了多少的permit，如果长时间没有尝试获取permit也不会无限积累，而是会使用maxPermits来限制。
  * 计算nextFreeTicketMicros参数
  * 计算了stableIntervalMicros参数（假设QPS为10，那么stableIntervalMicros = 1000ms / 10 = 100ms）。
  * 然后调用了SmoothBursty的doSetRate方法，主要是重新设置了maxPermits、storedPermits属性。
从上面可以看出，Rate Limiter的create方法主要是将相关属性进行初始化。在初始化之后，我们主要看两个关键的方法，一个时acquire方法，另一个是tryAcquire方法。

## acquire
 acquire方法在获取permit的同时，如果此次无法获取permit，会执行休眠，待休眠结束后，返回主线程继续执行。
```java
public class RateLimiter {
  /**
   * 这个方法每次都获取一个permit。
   */
  public double acquire() {
    return acquire(1);
  }

  public double acquire(int permits) {
    // 获取需要等待的时间
    long microsToWait = reserve(permits);
    // 调用SleepingStopWatch的不可中断方法实现线程休眠时间
    stopwatch.sleepMicrosUninterruptibly(microsToWait);
    // 返回
    return 1.0 * microsToWait / SECONDS.toMicros(1L);
  }
  final long reserve(int permits) {
    checkPermits(permits);
    synchronized (mutex()) {
      return reserveAndGetWaitLength(permits, stopwatch.readMicros());
    }
  }

  final long reserveAndGetWaitLength(int permits, long nowMicros) {
    // 返回的是上一次请求时计算好的下一次能够获取permit的时间
    // 这是一个时段，表明距离上一次获取permit之后，还需要等待的时长
    long momentAvailable = reserveEarliestAvailable(permits, nowMicros);
    // 如果nowMicros（当前时间距离上一次请求获取permit的时间差）< momentAvailable，表明需要等待。
    // 如果nowMicros ≥ momentAvailable，不需要等待，直接返回0.
    return max(momentAvailable - nowMicros, 0);
  }


}
```
调用SmoothRateLimiter类的reserveEarliestAvailable方法
```java
abstract class SmoothRateLimiter extends RateLimiter {
    @Override
  final long reserveEarliestAvailable(int requiredPermits, long nowMicros) {
    // 设置storedPermits和nextFreeTicketMicros值
    resync(nowMicros);
    long returnValue = nextFreeTicketMicros;
    // 如果requiredPermits（需要的permit）大于storedPermits（当前可能获取到最大的permits），
    // 那么就只能获取storedPermits个permit。这里有一个疑问如果
    double storedPermitsToSpend = min(requiredPermits, this.storedPermits);
    // 如果requiredPermits <= storedPermits, 那freshPermits = 0
    // 如果如果requiredPermits > storedPermits，那freshPermits > 0
    double freshPermits = requiredPermits - storedPermitsToSpend;
    // 如果实际创建的RateLimiter实现类是SmoothBursty，那么storedPermitsToWaitTime返回0
    // 如果实际创建的RateLimiter实现类是SmoothWarmingUp（解读待定）
    // 这里就可以计算下一次需要等待的时间
    long waitMicros =
        storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend)
            + (long) (freshPermits * stableIntervalMicros);
    // 计算下一次能够拿到permit的时间戳，如果这一次的requiredPermits比实际上storedPermits大，那么相当于
    // 这一次透支了下一次能够拿到的permit的时间
    this.nextFreeTicketMicros = LongMath.saturatedAdd(nextFreeTicketMicros, waitMicros);
    this.storedPermits -= storedPermitsToSpend;
    return returnValue;
  }
}
```
详细总结一下流程
* 调用RateLimiter的acquire方法，每次都会像RateLimiter申请一个permit，在申请permit之前，会调用SleepingStopWatch的readMicors方法计算本次请求距离RateLimiter创建时的时间，在程序中是nowMicros这个参数表征本次请求时间戳与RateLimiter创建时的时间差。调用SmoothRateLimiter的reserveEarliestAvailable方法来更新storedPerimits、nextFreeTicketMicros参数以及本次请求之后线程需要休眠的时长。
* 来更新storedPermit参数和nextFreeTicketMicros参数
  * storePermits参数表示RateLimiter在没有请求时能够积累的permit，
  * nextFreeTicketMicros参数表示上一次请求之后下一次请求的时刻（这个时刻是一个相对的时间，相对于RateLimiter创建时刻而言），下一次请求在这个之前都是会被阻塞（调用acquire方法会阻塞）或直接返回false（调用他tryAcquire方法）。
  * 如果nowMicros大于nextFreeTicketMicros，首先需要计算原本下次请求（也就是本次请求）之间有没有累积permit，主要计算方式是
  $$ storedPermits = (nowMicros - nextTicketMicros) / stableIntervalMicros $$
* 首先计算本次需要消耗的permit的数量，
  * 如果requiredPermits < storedPermts，那么此时需要消耗的permit的数量比实际上存储的permit的数量要小，那本次获取permit的数量之后还有余额。
  * 如果requiredPermits ≥ storedPermits，此时表明之前存储的permit时间不足了，此时storedPermits被消耗之后会置0，而不会是一个负数。这意味着你这一次超额获取permit之后，下一次获取permit需要等待的时间会变长（相当于信用卡超额消费了，下一次还），这个功能借用另一个参数nextFreeTicketMicros实现
* 然后计算nextFreeTicketMicros
  * 如果requiredPermits < storedPermits，那么下一次获取permit的时间间隔nextFreeTicketMicros其实就等于nowMicros。
  * 如果requiredPermits ≥ storedPerimits, 相当于本次超额申请了permit，因为在初始化RateLimiter时，初始化了获取一个permit的时间间隔参数stableIntervalMicros。因此this.nextFreeTicketMicros = nextFreeTicketMicros + (requiredPermits - storedPermits) * stableIntervalMicros;
* 将本次获取permit和上一次获取permit的时间差(nowMicros)，与上一次计算的nextFreeTicketMicros之间的差值，从而计算本次需要线程休眠的时长。
  * 如果nowMicros ≥ nextFreeTicketMicros，那么说明本次获取permit的间隔已经满足条件，不用执行线程休眠；
  * 如果nowMicros < nextFreeTicketMicros，说明本次获取permit的间隔太快了，需要等待一段时间，执行县城休眠。


## tryAcquire
这个方法会会申请permit的数量，如果能够获取足够多的

```java
public class RateLimiter {
  public boolean tryAcquire(long timeout, TimeUnit unit) {
    return tryAcquire(1, timeout, unit);
  }
  public boolean tryAcquire(int permits) {
    return tryAcquire(permits, 0, MICROSECONDS);
  }

  public boolean tryAcquire() {
    return tryAcquire(1, 0, MICROSECONDS);
  }
  public boolean tryAcquire(int permits, long timeout, TimeUnit unit) {
    long timeoutMicros = max(unit.toMicros(timeout), 0);
    // 校验参数，不是关键流程
    checkPermits(permits);
    long microsToWait;
    // 同步获取锁
    synchronized (mutex()) {
      // 上一次获取permit到这一次获取permit的时间差
      long nowMicros = stopwatch.readMicros();
      // 如果无法获取到permit则返回false，如果能够获取到足够的permit，这里会返回true，那么就会执行下面睡眠方法
      if (!canAcquire(nowMicros, timeoutMicros)) {
        return false;
      } else {
        // 如果获取到了，那么就需要更新等待的时间，这个方法和acquire一样的逻辑
        microsToWait = reserveAndGetWaitLength(permits, nowMicros);
      }
    }
    stopwatch.sleepMicrosUninterruptibly(microsToWait);
    return true;
  }
  private boolean canAcquire(long nowMicros, long timeoutMicros) {
    return queryEarliestAvailable(nowMicros) - timeoutMicros <= nowMicros;
  }

  // 这个方法主要返回需要等待的时间，同时重置storedPermits和nextFreeTicketMicros参数
  final long reserveAndGetWaitLength(int permits, long nowMicros) {
    long momentAvailable = reserveEarliestAvailable(permits, nowMicros);
    return max(momentAvailable - nowMicros, 0);
  }
}
```
从上面的的代码我们可以看到如果调用canAcquire方法返回false，那么就会直接返回；如果方法返回true，那么就会更新nextFreeTicketMicros和storedPermits。可以看一下看一下canAcquire方法。
```java
abstract class SmoothRateLimiter extends RateLimiter {
  @Override
  final long queryEarliestAvailable(long nowMicros) {
    return nextFreeTicketMicros;
  }
  final long reserveEarliestAvailable(int requiredPermits, long nowMicros) {
    // 设置storedPermits和nextFreeTicketMicros值
    resync(nowMicros);
    long returnValue = nextFreeTicketMicros;
    // 如果requiredPermits（需要的permit）大于storedPermits（当前可能获取到最大的permits），
    // 那么就只能获取storedPermits个permit。这里有一个疑问如果
    double storedPermitsToSpend = min(requiredPermits, this.storedPermits);
    // 如果requiredPermits <= storedPermits, 那freshPermits = 0
    // 如果如果requiredPermits > storedPermits，那freshPermits > 0
    double freshPermits = requiredPermits - storedPermitsToSpend;
    // 如果实际创建的RateLimiter实现类是SmoothBursty，那么storedPermitsToWaitTime返回0
    // 如果实际创建的RateLimiter实现类是SmoothWarmingUp（解读待定）
    // 这里就可以计算下一次需要等待的时间
    long waitMicros =
        storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend)
            + (long) (freshPermits * stableIntervalMicros);
    // 计算下一次能够拿到permit的时间戳，如果这一次的requiredPermits比实际上storedPermits大，那么相当于
    // 这一次透支了下一次能够拿到的permit的时间
    this.nextFreeTicketMicros = LongMath.saturatedAdd(nextFreeTicketMicros, waitMicros);
    this.storedPermits -= storedPermitsToSpend;
    return returnValue;
  }
  
  void resync(long nowMicros) {
    // if nextFreeTicket is in the past, resync to now
    if (nowMicros > nextFreeTicketMicros) {
      // 更新storedPermits，因为如果nowMicros > nowMicros，那么此时就需要看看是不是增加storedPermits的值
      double newPermits = (nowMicros - nextFreeTicketMicros) / coolDownIntervalMicros();
      storedPermits = min(maxPermits, storedPermits + newPermits);
      // 这里把下一次
      nextFreeTicketMicros = nowMicros;
    }
  }
}
```
这个方法相对于来说比较简单，就是获取上一次请求时设置的下一次请求的时间间隔。这个canAcquire方法与nowMicors参数没有关系，就是返回nextFreeTicketMicros。但是这个方法主要的作用时。

## 使用案例

### tryAcquire使用用例

```java
@Slf4j
public class RateLimiterTest extends TestCase {
    public void testTryAcquire() {
        RateLimiter rateLimiter = RateLimiter.create(10);
        log.info("create rate limiter finished , storedPermits::{}, maxPermits::{}, stableIntervalMicros::{}, nextFreeTicketMicros::{}",
                rateLimiter.getStoredPermits(), rateLimiter.getMaxPermits(), rateLimiter.getStableIntervalMicros(), rateLimiter.getNextFreeTicketMicros());
        boolean flag = rateLimiter.tryAcquire(5);
        log.info("get 8 permit flag:{}, storedPermits::{}, nextFreeTicketMicros::{} ", flag, rateLimiter.getStoredPermits(), rateLimiter.getNextFreeTicketMicros());
        sleep(500);
        flag = rateLimiter.tryAcquire(3);
        log.info("get 3 permit flag:{}, storedPermits::{}, nextFreeTicketMicros::{} ", flag, rateLimiter.getStoredPermits(), rateLimiter.getNextFreeTicketMicros());

    }
}
```
输出结果
```bash
10:44:44.634 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute resync method, lastFreeTicketMicros::195, nextFreeTicketMicros::195, coolDownIntervalMicros::0.0, storedPermits::0.0
10:44:44.638 [main] INFO ratelimiter.RateLimiterTest - create rate limiter finished , storedPermits::0.0, maxPermits::10.0, stableIntervalMicros::100000.0, nextFreeTicketMicros::195
10:44:44.640 [main] INFO com.zbt.ratelimiter.RateLimiter - the nowMicros is::6661
10:44:44.640 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute resync method, lastFreeTicketMicros::6661, nextFreeTicketMicros::6661, coolDownIntervalMicros::100000.0, storedPermits::0.06466
10:44:44.641 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute reserveEarliestAvailable method, lastFreeTicketMicros::6661, nextFreeTicketMicros::500195, waitMicros::493534
10:44:44.641 [main] INFO ratelimiter.RateLimiterTest - get 8 permit flag:true, storedPermits::0.0, nextFreeTicketMicros::500195 
10:44:45.146 [main] INFO com.zbt.ratelimiter.RateLimiter - the nowMicros is::512674
10:44:45.146 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute resync method, lastFreeTicketMicros::512674, nextFreeTicketMicros::512674, coolDownIntervalMicros::100000.0, storedPermits::0.12479
10:44:45.146 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute reserveEarliestAvailable method, lastFreeTicketMicros::512674, nextFreeTicketMicros::800195, waitMicros::287521
10:44:45.146 [main] INFO ratelimiter.RateLimiterTest - get 3 permit flag:true, storedPermits::0.0, nextFreeTicketMicros::800195 
```

### acquire使用案例

```java
@Slf4j
public class RateLimiterTest extends TestCase {

    public void testAcquire() {
        RateLimiter rateLimiter = RateLimiter.create(10);
        log.info("create rate limiter finished , storedPermits::{}, maxPermits::{}, stableIntervalMicros::{}, nextFreeTicketMicros::{}",
                rateLimiter.getStoredPermits(), rateLimiter.getMaxPermits(), rateLimiter.getStableIntervalMicros(), rateLimiter.getNextFreeTicketMicros());
        double time = rateLimiter.acquire(2);
        log.info("get 2 permit cost:{}, storedPermits::{}, nextFreeTicketMicros::{} ", time, rateLimiter.getStoredPermits(), rateLimiter.getNextFreeTicketMicros());

        time = rateLimiter.acquire(4);
        log.info("get 4 permit cost:{}, storedPermits::{}, nextFreeTicketMicros::{} ", time, rateLimiter.getStoredPermits(), rateLimiter.getNextFreeTicketMicros());

    }
}
```
输出结果
```bash
10:46:59.899 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute resync method, lastFreeTicketMicros::367, nextFreeTicketMicros::367, coolDownIntervalMicros::0.0, storedPermits::0.0
10:46:59.906 [main] INFO ratelimiter.RateLimiterTest - create rate limiter finished , storedPermits::0.0, maxPermits::10.0, stableIntervalMicros::100000.0, nextFreeTicketMicros::367
10:46:59.908 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute resync method, lastFreeTicketMicros::11691, nextFreeTicketMicros::11691, coolDownIntervalMicros::100000.0, storedPermits::0.11324
10:46:59.911 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute reserveEarliestAvailable method, lastFreeTicketMicros::11691, nextFreeTicketMicros::200367, waitMicros::188676
10:46:59.911 [main] INFO ratelimiter.RateLimiterTest - get 2 permit cost:0.0, storedPermits::0.0, nextFreeTicketMicros::200367 
10:46:59.911 [main] INFO com.zbt.ratelimiter.SmoothRateLimiter - execute reserveEarliestAvailable method, lastFreeTicketMicros::200367, nextFreeTicketMicros::600367, waitMicros::400000
10:47:00.098 [main] INFO ratelimiter.RateLimiterTest - get 4 permit cost:0.185836, storedPermits::0.0, nextFreeTicketMicros::600367 
```