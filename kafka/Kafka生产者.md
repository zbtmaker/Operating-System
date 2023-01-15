# Kafka生产者

## Kafka 生产者参数
关于buffer.memory这个参数的解答，我们的数据首先是放在缓冲池里面，
* 那么这个缓冲池的结构是什么样子的？
* 我们如何知道消费的速度，消费的方式是怎样的，是单线程消费还是多线程消费呢？
* 如果我们写入的数据是在缓冲池里面，那么我们如果future.get()同步等待结果是不是很慢呢？
* 根据上一步提出的问题，我们什么场景应该使用同步写入，什么时候应该使用异步写入呢？
* 如果我们设置的buffer.memory特别大，生产者是单线程写入的，此时又因为同步等待结果什么时候开始发送呢？
### batch.size和linger.ms
* batch.size：如果生产者写入的消息量达到了batch.size配置的值，比方说1024Byte，那么此时sender线程就会开始向broker多条消息通过打包的方式一次性传递过去。我们前面有一个疑问，如果producer是单线程且是同步写入的（将消息写入到缓冲池后会同步等待结果）,那么因为一个producer一次生产的消息无法满足batch.size的要求，那么这个时候是不是就不发送了，让producer这个生产者一直等待呢？
* linger.ms：为了解决上面因为batch.size配置的值过大，导致单线程同步写入一直阻塞的问题，Kafka的设计者通过配置linger.ms这个参数来解决问题，linger.ms表示Kafka会记录首次向缓冲池写入数据的时间，如果过了10ms（假如linger.ms=10），这个batch.size还是没有满足，此时sender线程也会将缓冲池中的消息发送给broker。

那么我们根据上面的猜测来写一个测试用例验证一下我们的猜想。
```java
package kafka;

import junit.framework.TestCase;
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.producer.*;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.Future;

/**
 * @author zoubaitao
 * date 2022/08/12
 */
@Slf4j
public class KafkaProducerTest extends TestCase {

    public void testBatchSize() {
        Map<String, Object> properties = new HashMap<>();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092,localhost:9093,localhost:9094");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        // 大约100KB
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, "100000");

        // 如果10秒内还没有写满100KBSender线程就发送消息至Broker
        properties.put(ProducerConfig.LINGER_MS_CONFIG, "10000");
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);
        String topic = "test";
        for (int i = 0; i < 10; i++) {
            long startTimer = System.currentTimeMillis();
            log.info("start send ::{}th message", i);
            ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topic, String.valueOf(i), String.valueOf(i));
            Future<RecordMetadata> future = kafkaProducer.send(producerRecord, (recordMetadata, e) -> log.info("the recordMetadata::{}", recordMetadata.offset()));
            try {
                RecordMetadata recordMetadata = future.get();
                log.info("receive message from send ::{}th message which cost time::{}", i, System.currentTimeMillis() - startTimer);
                if (recordMetadata == null) {
                    log.error("wait kafka send message result is null");
                }
            } catch (Exception ex) {
                log.error("wait kafka send message result error", ex);
            }

        }
        try {
            Thread.sleep(5000);
        } catch (Exception ex) {
            log.error("thread sleep error,", ex);
        }
    }
}
```

我们来看一下日志打印
```bash
23:53:02.104 [main] INFO kafka.KafkaProducerTest - start send ::1th message
23:53:12.105 [kafka-producer-network-thread | producer-1] DEBUG org.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Sending PRODUCE request with header RequestHeader(apiKey=PRODUCE, apiVersion=9, clientId=producer-1, correlationId=4) and timeout 30000 to node 0: {acks=-1,timeout=30000,partitionSizes=[test-3=70]}
23:53:12.109 [kafka-producer-network-thread | producer-1] DEBUG org.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Received PRODUCE response from node 0 for request with header RequestHeader(apiKey=PRODUCE, apiVersion=9, clientId=producer-1, correlationId=4): ProduceResponseData(responses=[TopicProduceResponse(name='test', partitionResponses=[PartitionProduceResponse(index=3, errorCode=0, baseOffset=1558, logAppendTimeMs=-1, logStartOffset=1547, recordErrors=[], errorMessage=null)])], throttleTimeMs=0)
23:53:12.109 [kafka-producer-network-thread | producer-1] INFO kafka.KafkaProducerTest - the recordMetadata::1558
23:53:12.109 [main] INFO kafka.KafkaProducerTest - receive message from send ::1th message which cost time::10005
23:53:12.109 [main] INFO kafka.KafkaProducerTest - start send ::2th message
23:53:22.110 [kafka-producer-network-thread | producer-1] DEBUG org.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Sending PRODUCE request with header RequestHeader(apiKey=PRODUCE, apiVersion=9, clientId=producer-1, correlationId=5) and timeout 30000 to node 0: {acks=-1,timeout=30000,partitionSizes=[test-0=70]}
23:53:22.113 [kafka-producer-network-thread | producer-1] DEBUG org.apache.kafka.clients.NetworkClient - [Producer clientId=producer-1] Received PRODUCE response from node 0 for request with header RequestHeader(apiKey=PRODUCE, apiVersion=9, clientId=producer-1, correlationId=5): ProduceResponseData(responses=[TopicProduceResponse(name='test', partitionResponses=[PartitionProduceResponse(index=0, errorCode=0, baseOffset=3284, logAppendTimeMs=-1, logStartOffset=3277, recordErrors=[], errorMessage=null)])], throttleTimeMs=0)
23:53:22.113 [kafka-producer-network-thread | producer-1] INFO kafka.KafkaProducerTest - the recordMetadata::3284
23:53:22.114 [main] INFO kafka.KafkaProducerTest - receive message from send ::2th message which cost time::10005
```
从上面的日志可以看到，如果发送消息的日志打印时间是23:53:02.104，但是因为我们设置的batch.size=1000000（也就是100KB左右），然后我们每次发送的消息不足100KB，因此这个需要等待缓冲池满，因此Kafka提供了另一个参数linger.ms，通过这个参数，如果缓冲池在一段时间，发送的数据无法满足缓冲池的大小，那么Kafka发送端会根据第一次收到消息开始计时，如果在linger.ms指定的时间内，发送的消息无法填满缓冲池，则会直接发送消息。

我们继续来解答batch这个参数的问题，当producer写入数据时，这个batch.size是针对所有同一个topic的所有partition而言，还是针对单个partition而言呢？我想这个应该是针对单个partition而言的。我们可以看一下源码
```java
    private Future<RecordMetadata> doSend(ProducerRecord<K, V> record, Callback callback) {
        TopicPartition tp = null;
        try {
            throwIfProducerClosed();
            // first make sure the metadata for the topic is available
            long nowMs = time.milliseconds();
            ClusterAndWaitTime clusterAndWaitTime;
            try {
                clusterAndWaitTime = waitOnMetadata(record.topic(), record.partition(), nowMs, maxBlockTimeMs);
            } catch (KafkaException e) {
                if (metadata.isClosed())
                    throw new KafkaException("Producer closed while send in progress", e);
                throw e;
            }
            nowMs += clusterAndWaitTime.waitedOnMetadataMs;
            long remainingWaitMs = Math.max(0, maxBlockTimeMs - clusterAndWaitTime.waitedOnMetadataMs);
            Cluster cluster = clusterAndWaitTime.cluster;
            // 使用指定的Key Serialization序列化工具序列化key
            byte[] serializedKey;
            try {
                serializedKey = keySerializer.serialize(record.topic(), record.headers(), record.key());
            } catch (ClassCastException cce) {
                throw new SerializationException("Can't convert key of class " + record.key().getClass().getName() +
                        " to class " + producerConfig.getClass(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG).getName() +
                        " specified in key.serializer", cce);
            }
            // 使用指定的Serialization序列化工具将value序列化为byte[]
            byte[] serializedValue;
            try {
                serializedValue = valueSerializer.serialize(record.topic(), record.headers(), record.value());
            } catch (ClassCastException cce) {
                throw new SerializationException("Can't convert value of class " + record.value().getClass().getName() +
                        " to class " + producerConfig.getClass(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG).getName() +
                        " specified in value.serializer", cce);
            }
            // 根据key 和value计算消息将要写入到哪个partition
            int partition = partition(record, serializedKey, serializedValue, cluster);
            tp = new TopicPartition(record.topic(), partition);

            setReadOnly(record.headers());
            Header[] headers = record.headers().toArray();

            int serializedSize = AbstractRecords.estimateSizeInBytesUpperBound(apiVersions.maxUsableProduceMagic(),
                    compressionType, serializedKey, serializedValue, headers);
            ensureValidRecordSize(serializedSize);
            long timestamp = record.timestamp() == null ? nowMs : record.timestamp();
            if (log.isTraceEnabled()) {
                log.trace("Attempting to append record {} with callback {} to topic {} partition {}", record, callback, record.topic(), partition);
            }
            // producer callback will make sure to call both 'callback' and interceptor callback
            Callback interceptCallback = new InterceptorCallback<>(callback, this.interceptors, tp);

            if (transactionManager != null && transactionManager.isTransactional()) {
                transactionManager.failIfNotReadyForSend();
            }
            // 将数据放入Accumulator中，我们可以看看Accumulator源码
            RecordAccumulator.RecordAppendResult result = accumulator.append(tp, timestamp, serializedKey,
                    serializedValue, headers, interceptCallback, remainingWaitMs, true, nowMs);

            if (result.abortForNewBatch) {
                int prevPartition = partition;
                partitioner.onNewBatch(record.topic(), cluster, prevPartition);
                partition = partition(record, serializedKey, serializedValue, cluster);
                tp = new TopicPartition(record.topic(), partition);
                if (log.isTraceEnabled()) {
                    log.trace("Retrying append due to new batch creation for topic {} partition {}. The old partition was {}", record.topic(), partition, prevPartition);
                }
                // producer callback will make sure to call both 'callback' and interceptor callback
                interceptCallback = new InterceptorCallback<>(callback, this.interceptors, tp);

                result = accumulator.append(tp, timestamp, serializedKey,
                    serializedValue, headers, interceptCallback, remainingWaitMs, false, nowMs);
            }

            if (transactionManager != null && transactionManager.isTransactional())
                transactionManager.maybeAddPartitionToTransaction(tp);

            if (result.batchIsFull || result.newBatchCreated) {
                log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
                this.sender.wakeup();
            }
            return result.future;
            // handling exceptions and record the errors;
            // for API exceptions return them in the future,
            // for other exceptions throw directly
        } catch (Exception e) {
            ...
        }
    }
```
doSend方法主要做了几件事
* 使用Key序列化工具序列化key 
* 使用Value序列化工具序列化value
* 根据key和broker集群信息计算消息将要写入到哪一个partition中
* 调用Accumulator对象的append方法，将消息写入到对应topic的partition的缓冲池中
* 

我们来看看缓冲消息的对象Accumulator对象
```java
public final class RecordAccumulator {

    private final Logger log;
    private volatile boolean closed;
    private final AtomicInteger flushesInProgress;
    private final AtomicInteger appendsInProgress;
    private final int batchSize;
    private final CompressionType compression;
    private final int lingerMs;
    private final long retryBackoffMs;
    private final int deliveryTimeoutMs;
    private final BufferPool free;
    private final Time time;
    private final ApiVersions apiVersions;
    private final ConcurrentMap<TopicPartition, Deque<ProducerBatch>> batches;
    private final IncompleteBatches incomplete;
    // The following variables are only accessed by the sender thread, so we don't need to protect them.
    private final Set<TopicPartition> muted;
    private int drainIndex;
    private final TransactionManager transactionManager;
    private long nextBatchExpiryTimeMs = Long.MAX_VALUE;
}
```
这里有一个batches变量用于存储每个TopicPartition（每一个topic和partition）对应要发送的消息，Dequeue这个数据结构是将元素插入到队列的尾部，出队列是从队列的头部，因此sender线程就是从队列的头部读取ProducerBatch数据。
我们看一下ProducerBatch对象
```java
public final class ProducerBatch {

    private static final Logger log = LoggerFactory.getLogger(ProducerBatch.class);

    private enum FinalState { ABORTED, FAILED, SUCCEEDED }

    final long createdMs;
    // 写入到哪一个Topic和Partition中
    final TopicPartition topicPartition;
    final ProduceRequestResult produceFuture;

    private final List<Thunk> thunks = new ArrayList<>();
    private final MemoryRecordsBuilder recordsBuilder;
    private final AtomicInteger attempts = new AtomicInteger(0);
    private final boolean isSplitBatch;
    private final AtomicReference<FinalState> finalState = new AtomicReference<>(null);

    int recordCount;
    int maxRecordSize;
    private long lastAttemptMs;
    private long lastAppendTime;
    private long drainedMs;
    private boolean retry;
    private boolean reopened;
}
```

具体的doSend方法
```java
public RecordAppendResult append(TopicPartition tp,
                                     long timestamp,
                                     byte[] key,
                                     byte[] value,
                                     Header[] headers,
                                     Callback callback,
                                     long maxTimeToBlock,
                                     boolean abortOnNewBatch,
                                     long nowMs) throws InterruptedException {
        // We keep track of the number of appending thread to make sure we do not miss batches in
        // abortIncompleteBatches().
        appendsInProgress.incrementAndGet();
        ByteBuffer buffer = null;
        if (headers == null) headers = Record.EMPTY_HEADERS;
        try {
            // 根据TopicPartition找到对应队列
            Deque<ProducerBatch> dq = getOrCreateDeque(tp);
            synchronized (dq) {
                if (closed)
                    throw new KafkaException("Producer closed while send in progress");
                RecordAppendResult appendResult = tryAppend(timestamp, key, value, headers, callback, dq, nowMs);
                if (appendResult != null)
                    return appendResult;
            }

            // we don't have an in-progress record batch try to allocate a new batch
            if (abortOnNewBatch) {
                // Return a result that will cause another call to append.
                return new RecordAppendResult(null, false, false, true);
            }

            byte maxUsableMagic = apiVersions.maxUsableProduceMagic();
            int size = Math.max(this.batchSize, AbstractRecords.estimateSizeInBytesUpperBound(maxUsableMagic, compression, key, value, headers));
            log.trace("Allocating a new {} byte message buffer for topic {} partition {} with remaining timeout {}ms", size, tp.topic(), tp.partition(), maxTimeToBlock);
            buffer = free.allocate(size, maxTimeToBlock);

            // Update the current time in case the buffer allocation blocked above.
            nowMs = time.milliseconds();
            synchronized (dq) {
                // Need to check if producer is closed again after grabbing the dequeue lock.
                if (closed)
                    throw new KafkaException("Producer closed while send in progress");
                
                RecordAppendResult appendResult = tryAppend(timestamp, key, value, headers, callback, dq, nowMs);
                if (appendResult != null) {
                    // Somebody else found us a batch, return the one we waited for! Hopefully this doesn't happen often...
                    return appendResult;
                }

                MemoryRecordsBuilder recordsBuilder = recordsBuilder(buffer, maxUsableMagic);
                ProducerBatch batch = new ProducerBatch(tp, recordsBuilder, nowMs);
                FutureRecordMetadata future = Objects.requireNonNull(batch.tryAppend(timestamp, key, value, headers,
                        callback, nowMs));

                dq.addLast(batch);
                incomplete.add(batch);

                // Don't deallocate this buffer in the finally block as it's being used in the record batch
                buffer = null;
                return new RecordAppendResult(future, dq.size() > 1 || batch.isFull(), true, false);
            }
        } finally {
            if (buffer != null)
                free.deallocate(buffer);
            appendsInProgress.decrementAndGet();
        }
    }
```
### buffer.memory和max.block.ms
消息写入到Kafka的服务器之前会在消息生产者的客户端给不同的分区申请batch.size大小的缓冲区，如果缓冲区较小，申请的batch.size也会受限。默认的batch.size的大小是16KB，而默认的buffer.memory的默认大小为32MB。我们这里使用多线程往Kafka里写数据看看什么时
### max.request.bytes
这里又引入了另一个参数max.request.bytes的使用，如果我们要往Kafka中写消息，如果我们要写入的一条消息的大小超出了max.request.bytes的大小，在发送消息时会报异常。因此，在发送消息之前，我们需要衡量发送单条消息的最大消息的大小。
```java
public class KafkaProducerTest extends TestCase {

    public void testMaxRequestSize() {
        Map<String, Object> properties = new HashMap<>();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092,localhost:9093,localhost:9094");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        // 大约100KB
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, "100000");

        // 如果10秒内还没有写满100KBSender线程就发送消息至Broker
        properties.put(ProducerConfig.LINGER_MS_CONFIG, "1");

        properties.put(ProducerConfig.MAX_REQUEST_SIZE_CONFIG,"1048576");

        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);
        String topic = "test";
        for (int i = 0; i < 10; i++) {
            long startTimer = System.currentTimeMillis();
            log.info("start send ::{}th message", i);
            byte[] arr = new byte[1048576];
            Arrays.fill(arr, (byte) 0);
            ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topic,String.valueOf(i) , Arrays.toString(arr));
            Future<RecordMetadata> future = kafkaProducer.send(producerRecord, (recordMetadata, e) -> log.info("the recordMetadata::{}", recordMetadata.offset()));
            try {
                RecordMetadata recordMetadata = future.get();
                log.info("receive message from send ::{}th message which cost time::{}", i, System.currentTimeMillis() - startTimer);
                if (recordMetadata == null) {
                    log.error("wait kafka send message result is null");
                }
            } catch (Exception ex) {
                log.error("wait kafka send message result error", ex);
            }
        }
        try {
            Thread.sleep(5000);
        } catch (Exception ex) {
            log.error("thread sleep error,", ex);
        }
    }
}
```
会打印日志
```bash
23:56:41.045 [main] DEBUG org.apache.kafka.clients.producer.KafkaProducer - [Producer clientId=producer-1] Exception occurred during message send:
org.apache.kafka.common.errors.RecordTooLargeException: The message is 3145817 bytes when serialized which is larger than 1048576, which is the value of the max.request.size configuration.

java.lang.NullPointerException
	at kafka.KafkaProducerTest.lambda$testMaxRequestSize$2(KafkaProducerTest.java:92)
	at org.apache.kafka.clients.producer.KafkaProducer.doSend(KafkaProducer.java:986)
	at org.apache.kafka.clients.producer.KafkaProducer.send(KafkaProducer.java:889)
	at kafka.KafkaProducerTest.testMaxRequestSize(KafkaProducerTest.java:92)
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
```