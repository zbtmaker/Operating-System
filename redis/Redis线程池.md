之前发送push的时候经常会出现没有节点可以获取，一直以为是Redis服务器没有线程池，但是今年有一个需求需要了解线程池的问题，因此需要取了解Redis线程池相关的事情。

```java
  @Deprecated
  public JedisClusterConnectionHandler(Set<HostAndPort> nodes,
      final JedisClientConfig seedNodesClientConfig,
      final GenericObjectPoolConfig<Jedis> poolConfig,
      final JedisClientConfig clusterNodesClientConfig) {
    this.cache = new JedisClusterInfoCache(poolConfig, clusterNodesClientConfig);
    initializeSlotsCache(nodes, seedNodesClientConfig);
  }
```

```java
    private void initializeSlotsCache(Set<HostAndPort> startNodes, JedisClientConfig clientConfig) {
        ArrayList<HostAndPort> startNodeList = new ArrayList<>(startNodes);
        Collections.shuffle(startNodeList);
        for (HostAndPort hostAndPort : startNodeList) {
            try (Jedis jedis = new Jedis(hostAndPort, clientConfig)) {
                cache.discoverClusterNodesAndSlots(jedis);
                return;
            } catch (JedisConnectionException e) {
                // try next nodes
            }
        }
    }
```

```java
    public void discoverClusterNodesAndSlots(Jedis jedis) {
        w.lock();

        try {
            reset();
            // 获取clusterSlots，这里会通过jedis的client请求slot信息
            List<Object> slots = jedis.clusterSlots();

            for (Object slotInfoObj : slots) {
                // 获取slot信息
                List<Object> slotInfo = (List<Object>) slotInfoObj;

                if (slotInfo.size() <= MASTER_NODE_INDEX) {
                    continue;
                }
                // 
                List<Integer> slotNums = getAssignedSlotArray(slotInfo);

                // hostInfos
                int size = slotInfo.size();
                for (int i = MASTER_NODE_INDEX; i < size; i++) {
                    List<Object> hostInfos = (List<Object>) slotInfo.get(i);
                    if (hostInfos.isEmpty()) {
                        continue;
                    }

                    HostAndPort targetNode = generateHostAndPort(hostInfos);
                    // 判断当前的targetNode是否已经创建了JedisPool
                    setupNodeIfNotExist(targetNode);
                    if (i == MASTER_NODE_INDEX) {
                        assignSlotsToNode(slotNums, targetNode);
                    }
                }
            }
        } finally {
            w.unlock();
        }
    }

    private List<Integer> getAssignedSlotArray(List<Object> slotInfo) {
        List<Integer> slotNums = new ArrayList<>();
        for (int slot = ((Long) slotInfo.get(0)).intValue(); slot <= ((Long) slotInfo.get(1)).intValue(); slot++) {
            slotNums.add(slot);
        }
        return slotNums;
    }

    private HostAndPort generateHostAndPort(List<Object> hostInfos) {
        String host = SafeEncoder.encode((byte[]) hostInfos.get(0));
        int port = ((Long) hostInfos.get(1)).intValue();
        return new HostAndPort(host, port);
    }

    public JedisPool setupNodeIfNotExist(final HostAndPort node) {
        w.lock();
        try {
            String nodeKey = getNodeKey(node);
            JedisPool existingPool = nodes.get(nodeKey);
            if (existingPool != null) return existingPool;
            // 为集群中每一个master节点都会创建一个JedisPool
            JedisPool nodePool = new JedisPool(poolConfig, node, clientConfig);
            nodes.put(nodeKey, nodePool);
            return nodePool;
        } finally {
            w.unlock();
        }
    }
```

## 测试

### redis server关停
```java
    public void test2() {
        GenericObjectPoolConfig<Jedis> poolConfig = new GenericObjectPoolConfig<>();
        poolConfig.setMaxTotal(4);
        poolConfig.setMaxWait(Duration.ofSeconds(5));
        JedisPool jedisPool = new JedisPool(poolConfig, "127.0.0.1", 6379);
        //向JedisPool借用8次连接，但是没有执行归还操作。
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.ping();
            Thread.sleep(10000);
            // 把redis-server关掉，此时会
            jedis.set("1","1");
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        jedisPool.getResource().ping();
    }
```
此时输出结果
```java
redis.clients.jedis.exceptions.JedisConnectionException: Unexpected end of stream.
	at redis.clients.jedis.util.RedisInputStream.ensureFill(RedisInputStream.java:202)
	at redis.clients.jedis.util.RedisInputStream.readByte(RedisInputStream.java:43)
	at redis.clients.jedis.Protocol.process(Protocol.java:162)
	at redis.clients.jedis.Protocol.read(Protocol.java:227)
	at redis.clients.jedis.Connection.readProtocolWithCheckingBroken(Connection.java:352)
	at redis.clients.jedis.Connection.getStatusCodeReply(Connection.java:270)
	at redis.clients.jedis.Jedis.set(Jedis.java:227)
	at redis.JedisTest.test2(JedisTest.java:48)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at junit.framework.TestCase.runTest(TestCase.java:177)
	at junit.framework.TestCase.runBare(TestCase.java:142)
	at junit.framework.TestResult$1.protect(TestResult.java:122)
	at junit.framework.TestResult.runProtected(TestResult.java:142)
	at junit.framework.TestResult.run(TestResult.java:125)
	at junit.framework.TestCase.run(TestCase.java:130)
	at junit.framework.TestSuite.runTest(TestSuite.java:241)
	at junit.framework.TestSuite.run(TestSuite.java:236)
	at org.junit.internal.runners.JUnit38ClassRunner.run(JUnit38ClassRunner.java:90)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)

redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool

	at redis.clients.jedis.util.Pool.getResource(Pool.java:84)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:370)
	at redis.JedisTest.test2(JedisTest.java:52)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at junit.framework.TestCase.runTest(TestCase.java:177)
	at junit.framework.TestCase.runBare(TestCase.java:142)
	at junit.framework.TestResult$1.protect(TestResult.java:122)
	at junit.framework.TestResult.runProtected(TestResult.java:142)
	at junit.framework.TestResult.run(TestResult.java:125)
	at junit.framework.TestCase.run(TestCase.java:130)
	at junit.framework.TestSuite.runTest(TestSuite.java:241)
	at junit.framework.TestSuite.run(TestSuite.java:236)
	at org.junit.internal.runners.JUnit38ClassRunner.run(JUnit38ClassRunner.java:90)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
Caused by: redis.clients.jedis.exceptions.JedisConnectionException: Failed to create socket.
	at redis.clients.jedis.DefaultJedisSocketFactory.createSocket(DefaultJedisSocketFactory.java:110)
	at redis.clients.jedis.Connection.connect(Connection.java:226)
	at redis.clients.jedis.BinaryClient.connect(BinaryClient.java:140)
	at redis.clients.jedis.BinaryJedis.connect(BinaryJedis.java:310)
	at redis.clients.jedis.BinaryJedis.initializeFromClientConfig(BinaryJedis.java:88)
	at redis.clients.jedis.BinaryJedis.<init>(BinaryJedis.java:293)
	at redis.clients.jedis.Jedis.<init>(Jedis.java:169)
	at redis.clients.jedis.JedisFactory.makeObject(JedisFactory.java:177)
	at org.apache.commons.pool2.impl.GenericObjectPool.create(GenericObjectPool.java:571)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:298)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:223)
	at redis.clients.jedis.util.Pool.getResource(Pool.java:75)
	... 20 more
Caused by: java.net.ConnectException: Connection refused (Connection refused)
	at java.net.PlainSocketImpl.socketConnect(Native Method)
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:476)
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:218)
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:200)
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:394)
	at java.net.Socket.connect(Socket.java:606)
	at redis.clients.jedis.DefaultJedisSocketFactory.createSocket(DefaultJedisSocketFactory.java:80)
	... 31 more
```

### 连接池不归还
```java
    public void test1() {
        GenericObjectPoolConfig<Jedis> poolConfig = new GenericObjectPoolConfig<>();
        poolConfig.setMaxTotal(4);
        poolConfig.setMaxWait(Duration.ofSeconds(5));
        JedisPool jedisPool = new JedisPool(poolConfig, "127.0.0.1", 6379);
        //向JedisPool借用8次连接，但是没有执行归还操作。
        for (int i = 0; i < 8; i++) {
            Jedis jedis = null;
            try {
                jedis = jedisPool.getResource();
                jedis.ping();
            } catch (Exception e) {
                log.error(e.getMessage(), e);
            }
        }
    }
```
Jedis Pool创建时会创建4个Socket链接，如果新来的一个客户端在5s内没有获取Jedis资源，就会抛出异常
```java
redis.clients.jedis.exceptions.JedisExhaustedPoolException: Could not get a resource since the pool is exhausted
	at redis.clients.jedis.util.Pool.getResource(Pool.java:78)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:370)
	at redis.JedisTest.test1(JedisTest.java:27)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at junit.framework.TestCase.runTest(TestCase.java:177)
	at junit.framework.TestCase.runBare(TestCase.java:142)
	at junit.framework.TestResult$1.protect(TestResult.java:122)
	at junit.framework.TestResult.runProtected(TestResult.java:142)
	at junit.framework.TestResult.run(TestResult.java:125)
	at junit.framework.TestCase.run(TestCase.java:130)
	at junit.framework.TestSuite.runTest(TestSuite.java:241)
	at junit.framework.TestSuite.run(TestSuite.java:236)
	at org.junit.internal.runners.JUnit38ClassRunner.run(JUnit38ClassRunner.java:90)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
Caused by: java.util.NoSuchElementException: Timeout waiting for idle object, borrowMaxWaitDuration=PT5S
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:312)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:223)
	at redis.clients.jedis.util.Pool.getResource(Pool.java:75)
	... 20 common frames omitted
10:28:20.535 [main] ERROR redis.JedisTest - Could not get a resource since the pool is exhausted
redis.clients.jedis.exceptions.JedisExhaustedPoolException: Could not get a resource since the pool is exhausted
	at redis.clients.jedis.util.Pool.getResource(Pool.java:78)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:370)
	at redis.JedisTest.test1(JedisTest.java:27)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at junit.framework.TestCase.runTest(TestCase.java:177)
	at junit.framework.TestCase.runBare(TestCase.java:142)
	at junit.framework.TestResult$1.protect(TestResult.java:122)
	at junit.framework.TestResult.runProtected(TestResult.java:142)
	at junit.framework.TestResult.run(TestResult.java:125)
	at junit.framework.TestCase.run(TestCase.java:130)
	at junit.framework.TestSuite.runTest(TestSuite.java:241)
	at junit.framework.TestSuite.run(TestSuite.java:236)
	at org.junit.internal.runners.JUnit38ClassRunner.run(JUnit38ClassRunner.java:90)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
Caused by: java.util.NoSuchElementException: Timeout waiting for idle object, borrowMaxWaitDuration=PT5S
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:312)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:223)
	at redis.clients.jedis.util.Pool.getResource(Pool.java:75)
	... 20 common frames omitted
10:28:25.536 [main] ERROR redis.JedisTest - Could not get a resource since the pool is exhausted
redis.clients.jedis.exceptions.JedisExhaustedPoolException: Could not get a resource since the pool is exhausted
	at redis.clients.jedis.util.Pool.getResource(Pool.java:78)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:370)
	at redis.JedisTest.test1(JedisTest.java:27)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at junit.framework.TestCase.runTest(TestCase.java:177)
	at junit.framework.TestCase.runBare(TestCase.java:142)
	at junit.framework.TestResult$1.protect(TestResult.java:122)
	at junit.framework.TestResult.runProtected(TestResult.java:142)
	at junit.framework.TestResult.run(TestResult.java:125)
	at junit.framework.TestCase.run(TestCase.java:130)
	at junit.framework.TestSuite.runTest(TestSuite.java:241)
	at junit.framework.TestSuite.run(TestSuite.java:236)
	at org.junit.internal.runners.JUnit38ClassRunner.run(JUnit38ClassRunner.java:90)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
Caused by: java.util.NoSuchElementException: Timeout waiting for idle object, borrowMaxWaitDuration=PT5S
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:312)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:223)
	at redis.clients.jedis.util.Pool.getResource(Pool.java:75)
	... 20 common frames omitted
10:28:30.542 [main] ERROR redis.JedisTest - Could not get a resource since the pool is exhausted
redis.clients.jedis.exceptions.JedisExhaustedPoolException: Could not get a resource since the pool is exhausted
	at redis.clients.jedis.util.Pool.getResource(Pool.java:78)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:370)
	at redis.JedisTest.test1(JedisTest.java:27)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at junit.framework.TestCase.runTest(TestCase.java:177)
	at junit.framework.TestCase.runBare(TestCase.java:142)
	at junit.framework.TestResult$1.protect(TestResult.java:122)
	at junit.framework.TestResult.runProtected(TestResult.java:142)
	at junit.framework.TestResult.run(TestResult.java:125)
	at junit.framework.TestCase.run(TestCase.java:130)
	at junit.framework.TestSuite.runTest(TestSuite.java:241)
	at junit.framework.TestSuite.run(TestSuite.java:236)
	at org.junit.internal.runners.JUnit38ClassRunner.run(JUnit38ClassRunner.java:90)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
Caused by: java.util.NoSuchElementException: Timeout waiting for idle object, borrowMaxWaitDuration=PT5S
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:312)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:223)
	at redis.clients.jedis.util.Pool.getResource(Pool.java:75)
	... 20 common frames omitted

redis.clients.jedis.exceptions.JedisExhaustedPoolException: Could not get a resource since the pool is exhausted

	at redis.clients.jedis.util.Pool.getResource(Pool.java:78)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:370)
	at redis.JedisTest.test1(JedisTest.java:33)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at junit.framework.TestCase.runTest(TestCase.java:177)
	at junit.framework.TestCase.runBare(TestCase.java:142)
	at junit.framework.TestResult$1.protect(TestResult.java:122)
	at junit.framework.TestResult.runProtected(TestResult.java:142)
	at junit.framework.TestResult.run(TestResult.java:125)
	at junit.framework.TestCase.run(TestCase.java:130)
	at junit.framework.TestSuite.runTest(TestSuite.java:241)
	at junit.framework.TestSuite.run(TestSuite.java:236)
	at org.junit.internal.runners.JUnit38ClassRunner.run(JUnit38ClassRunner.java:90)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:69)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:220)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:53)
Caused by: java.util.NoSuchElementException: Timeout waiting for idle object, borrowMaxWaitDuration=PT5S
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:312)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:223)
	at redis.clients.jedis.util.Pool.getResource(Pool.java:75)
	... 20 more
```