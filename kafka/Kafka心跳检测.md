# Kafka心跳检测

## 关于Time和Timer
 Time是一个接口，主要实现者是SystemTime，主要提供的功能是获取服务器系统时间戳
 ```java
public interface Time {

    Time SYSTEM = new SystemTime();

    /**
     * Returns the current time in milliseconds.
     */
    long milliseconds();
 
    default long hiResClockMs() {
        return TimeUnit.NANOSECONDS.toMillis(nanoseconds());
    }

    long nanoseconds();

    void sleep(long ms);

    void waitObject(Object obj, Supplier<Boolean> condition, long deadlineMs) throws InterruptedException;
 
    default Timer timer(long timeoutMs) {
        return new Timer(this, timeoutMs);
    }
 
    default Timer timer(Duration timeout) {
        return timer(timeout.toMillis());
    }
}
```

Timer这个类主要是为心跳检测（heartbeat.timeout.ms）、会话检测（session.timeout.ms）、消费耗时检测（max.poll.timeout.ms）
```java
public class Timer {
    private final Time time;
    private long startMs;
    private long currentTimeMs;
    private long deadlineMs;
    private long timeoutMs;

    Timer(Time time, long timeoutMs) {
        this.time = time;
        update();
        reset(timeoutMs);
    }
    // 判断当前时间是否比deadlineMs要大，如果大的话说明当前会话已经过期了
    public boolean isExpired() {
        return currentTimeMs >= deadlineMs;
    }
 
    public boolean notExpired() {
        return !isExpired();
    }
 
    public void updateAndReset(long timeoutMs) {
        update();
        reset(timeoutMs);
    }
    /**
     * 这里的重置方案，就是将currentTimeMs重置为系统当前时间
     */
    public void reset(long timeoutMs) {
        if (timeoutMs < 0)
            throw new IllegalArgumentException("Invalid negative timeout " + timeoutMs);

        this.timeoutMs = timeoutMs;
        this.startMs = this.currentTimeMs;

        if (currentTimeMs > Long.MAX_VALUE - timeoutMs)
            this.deadlineMs = Long.MAX_VALUE;
        else
            this.deadlineMs = currentTimeMs + timeoutMs;
    }
 
    public void resetDeadline(long deadlineMs) {
        if (deadlineMs < 0)
            throw new IllegalArgumentException("Invalid negative deadline " + deadlineMs);

        this.timeoutMs = Math.max(0, deadlineMs - this.currentTimeMs);
        this.startMs = this.currentTimeMs;
        this.deadlineMs = deadlineMs;
    }
 
    public void update() {
        update(time.milliseconds());
    }

    public void update(long currentTimeMs) {
        this.currentTimeMs = Math.max(currentTimeMs, this.currentTimeMs);
    }

    public long remainingMs() {
        // 这里是看这个deadline和currentTimeMs进行比较，如果deadlineMs < currentTimeMs，表明这一次的会话已经超出时间了
        return Math.max(0, deadlineMs - currentTimeMs);
    }
 
    public void sleep(long durationMs) {
        long sleepDurationMs = Math.min(durationMs, remainingMs());
        time.sleep(sleepDurationMs);
        update();
    }
}

```

HeartBeat
* heartbeatTimer：保存消费者的配置项heartbeat.interval.ms配置
* sessionTimer：保存session.timeout.ms配置
* poolTimer：max.poll.interval.ms配置

```java
public final class Heartbeat {
    private final int maxPollIntervalMs;
    private final GroupRebalanceConfig rebalanceConfig;
    private final Time time;
    private final Timer heartbeatTimer;
    private final Timer sessionTimer;
    private final Timer pollTimer;
    private final Logger log;

    private volatile long lastHeartbeatSend = 0L;
    private volatile boolean heartbeatInFlight = false;

    public Heartbeat(GroupRebalanceConfig config,
                     Time time) {
        if (config.heartbeatIntervalMs >= config.sessionTimeoutMs)
            throw new IllegalArgumentException("Heartbeat must be set lower than the session timeout");
        this.rebalanceConfig = config;
        this.time = time;
        this.heartbeatTimer = time.timer(config.heartbeatIntervalMs);
        this.sessionTimer = time.timer(config.sessionTimeoutMs);
        this.maxPollIntervalMs = config.rebalanceTimeoutMs;
        this.pollTimer = time.timer(maxPollIntervalMs);

        final LogContext logContext = new LogContext("[Heartbeat groupID=" + config.groupId + "] ");
        this.log = logContext.logger(getClass());
    }

    private void update(long now) {
        heartbeatTimer.update(now);
        sessionTimer.update(now);
        pollTimer.update(now);
    }

    public void poll(long now) {
        update(now);
        pollTimer.reset(maxPollIntervalMs);
    }

    boolean hasInflight() {
        return heartbeatInFlight;
    }

    void sentHeartbeat(long now) {
        lastHeartbeatSend = now;
        heartbeatInFlight = true;
        update(now);
        heartbeatTimer.reset(rebalanceConfig.heartbeatIntervalMs);

        if (log.isTraceEnabled()) {
            log.trace("Sending heartbeat request with {}ms remaining on timer", heartbeatTimer.remainingMs());
        }
    }

    void failHeartbeat() {
        update(time.milliseconds());
        heartbeatInFlight = false;
        heartbeatTimer.reset(rebalanceConfig.retryBackoffMs);

        log.trace("Heartbeat failed, reset the timer to {}ms remaining", heartbeatTimer.remainingMs());
    }

    void receiveHeartbeat() {
        update(time.milliseconds());
        heartbeatInFlight = false;
        sessionTimer.reset(rebalanceConfig.sessionTimeoutMs);
    }

    boolean shouldHeartbeat(long now) {
        update(now);
        return heartbeatTimer.isExpired();
    }
    
    long lastHeartbeatSend() {
        return this.lastHeartbeatSend;
    }

    long timeToNextHeartbeat(long now) {
        update(now);
        return heartbeatTimer.remainingMs();
    }

    boolean sessionTimeoutExpired(long now) {
        update(now);
        return sessionTimer.isExpired();
    }

    void resetTimeouts() {
        update(time.milliseconds());
        sessionTimer.reset(rebalanceConfig.sessionTimeoutMs);
        pollTimer.reset(maxPollIntervalMs);
        heartbeatTimer.reset(rebalanceConfig.heartbeatIntervalMs);
    }

    void resetSessionTimeout() {
        update(time.milliseconds());
        sessionTimer.reset(rebalanceConfig.sessionTimeoutMs);
    }

    boolean pollTimeoutExpired(long now) {
        update(now);
        return pollTimer.isExpired();
    }

    long lastPollTime() {
        return pollTimer.currentTimeMs();
    }
}

```

## 心跳检测
在AbstractCoordinator这个类里面有个一个HeartbeatThread，这个线程主要负责Kafka Consumer心跳保活检测，主要逻辑如下
```java
        public void run() {
            try {
                log.debug("Heartbeat thread started");
                while (true) {
                    synchronized (AbstractCoordinator.this) {
                        if (closed)
                            return;

                        if (!enabled) {
                            AbstractCoordinator.this.wait();
                            continue;
                        }

                        // we do not need to heartbeat we are not part of a group yet;
                        // also if we already have fatal error, the client will be
                        // crashed soon, hence we do not need to continue heartbeating either
                        if (state.hasNotJoinedGroup() || hasFailed()) {
                            disable();
                            continue;
                        }

                        client.pollNoWakeup();
                        long now = time.milliseconds();

                        if (coordinatorUnknown()) {
                            if (findCoordinatorFuture != null) {
                                // clear the future so that after the backoff, if the hb still sees coordinator unknown in
                                // the next iteration it will try to re-discover the coordinator in case the main thread cannot
                                clearFindCoordinatorFuture();

                                // backoff properly
                                AbstractCoordinator.this.wait(rebalanceConfig.retryBackoffMs);
                            } else {
                                lookupCoordinator();
                            }
                        } else if (heartbeat.sessionTimeoutExpired(now)) {
                            // the session timeout has expired without seeing a successful heartbeat, so we should
                            // probably make sure the coordinator is still healthy.
                            markCoordinatorUnknown("session timed out without receiving a "
                                    + "heartbeat response");
                        } else if (heartbeat.pollTimeoutExpired(now)) {
                            // the poll timeout has expired, which means that the foreground thread has stalled
                            // in between calls to poll().
                            log.warn("consumer poll timeout has expired. This means the time between subsequent calls to poll() " +
                                "was longer than the configured max.poll.interval.ms, which typically implies that " +
                                "the poll loop is spending too much time processing messages. You can address this " +
                                "either by increasing max.poll.interval.ms or by reducing the maximum size of batches " +
                                "returned in poll() with max.poll.records.");

                            maybeLeaveGroup("consumer poll timeout has expired.");
                        } else if (!heartbeat.shouldHeartbeat(now)) {
                            // poll again after waiting for the retry backoff in case the heartbeat failed or the
                            // coordinator disconnected
                            AbstractCoordinator.this.wait(rebalanceConfig.retryBackoffMs);
                        } else {
                            heartbeat.sentHeartbeat(now);
                            final RequestFuture<Void> heartbeatFuture = sendHeartbeatRequest();
                            heartbeatFuture.addListener(new RequestFutureListener<Void>() {
                                @Override
                                public void onSuccess(Void value) {
                                    synchronized (AbstractCoordinator.this) {
                                        heartbeat.receiveHeartbeat();
                                    }
                                }

                                @Override
                                public void onFailure(RuntimeException e) {
                                    synchronized (AbstractCoordinator.this) {
                                        if (e instanceof RebalanceInProgressException) {
                                            // it is valid to continue heartbeating while the group is rebalancing. This
                                            // ensures that the coordinator keeps the member in the group for as long
                                            // as the duration of the rebalance timeout. If we stop sending heartbeats,
                                            // however, then the session timeout may expire before we can rejoin.
                                            heartbeat.receiveHeartbeat();
                                        } else if (e instanceof FencedInstanceIdException) {
                                            log.error("Caught fenced group.instance.id {} error in heartbeat thread", rebalanceConfig.groupInstanceId);
                                            heartbeatThread.failed.set(e);
                                        } else {
                                            heartbeat.failHeartbeat();
                                            // wake up the thread if it's sleeping to reschedule the heartbeat
                                            AbstractCoordinator.this.notify();
                                        }
                                    }
                                }
                            });
                        }
                    }
                }
            } catch (AuthenticationException e) {
                log.error("An authentication error occurred in the heartbeat thread", e);
                this.failed.set(e);
            } catch (GroupAuthorizationException e) {
                log.error("A group authorization error occurred in the heartbeat thread", e);
                this.failed.set(e);
            } catch (InterruptedException | InterruptException e) {
                Thread.interrupted();
                log.error("Unexpected interrupt received in heartbeat thread", e);
                this.failed.set(new RuntimeException(e));
            } catch (Throwable e) {
                log.error("Heartbeat thread failed due to unexpected error", e);
                if (e instanceof RuntimeException)
                    this.failed.set((RuntimeException) e);
                else
                    this.failed.set(new RuntimeException(e));
            } finally {
                log.debug("Heartbeat thread has closed");
            }
        }
```

## 参考
[Kafka学习笔记](https://zhmin.github.io/categories/kafka/)