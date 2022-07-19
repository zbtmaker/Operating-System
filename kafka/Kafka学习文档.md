# Kafka安装
## 一、使用HomeBrew安装Kafka
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install kafka
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/op
######################################################################## 100.0%
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/ca
######################################################################## 100.0%
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/op
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/manifests/1.1.1o
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/blobs/sha256:630f15
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sh
######################################################################## 100.0%
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/ka
######################################################################## 100.0%
==> Installing dependencies for kafka: openjdk, ca-certificates and openssl@1.1
==> Installing kafka dependency: openjdk
==> Pouring openjdk-18.0.1.1.monterey.bottle.tar.gz
🍺  /usr/local/Cellar/openjdk/18.0.1.1: 641 files, 307.7MB
==> Installing kafka dependency: ca-certificates
==> Pouring ca-certificates-2022-04-26.all.bottle.tar.gz
==> Regenerating CA certificate bundle from keychain, this may take a while...
🍺  /usr/local/Cellar/ca-certificates/2022-04-26: 3 files, 215.5KB
==> Installing kafka dependency: openssl@1.1
==> Pouring openssl@1.1-1.1.1o.monterey.bottle.tar.gz
Error: No such file or directory @ rb_sysopen - /Users/zoubaitao/Library/Caches/Homebrew/downloads/3465cbaa9e4d6661e5b5a1a02f37a8e7d41345392c56dd8d51688c656a554bb1--openssl@1.1-1.1.1o.monterey.bottle.tar.gz
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install openssl
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/openssl%403-3.0.3.monterey.bottle.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/3/manifests/3.0.3
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/3/blobs/sha256:044032dcdffb16c0bf7949fdf847f5f957e727bef1f982d8a7f156f727c81ad5
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:044032dcdffb16c0bf7949fdf847f5f957e727bef1f982d8a7f156f727c81ad5?se=2022-07-03T03%3A15%3A00Z&sig=4nfhjG6SMSzOW2AkNoPpIfRwignkXVvZkW9n8nd45rk%3D&sp=r&spr=https&sr=b&sv=
######################                                                    31.5%^C
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install openssl@1.1
openssl@1.1  is already installed but outdated (so it will be upgraded).
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/openssl%401.1-1.1.1o.monterey.bottle.tar.gz
curl: (22) The requested URL returned error: 404                              

Warning: Bottle missing, falling back to the default domain...
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/manifests/1.1.1o
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/75e64159933cb7401740964ef8681602932bd4c67aade21021fd522238a05e20--openssl@1.1-1.1.1o.bottle_manifest.json
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/1.1/blobs/sha256:630f1510f71f8ad7f9521d3e7371bee08f1956544a48d9796fb5d1fab4058581
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/9814e80d4f5817f82a97364dbbe57b01123e34f6fa4c196e563d48d871530ecd--openssl@1.1--1.1.1o.monterey.bottle.tar.gz
==> Pouring openssl@1.1--1.1.1o.monterey.bottle.tar.gz
==> Caveats
A CA file has been bootstrapped using certificates from the system
keychain. To add additional certificates, place .pem files in
  /usr/local/etc/openssl@1.1/certs

and run
  /usr/local/opt/openssl@1.1/bin/c_rehash

openssl@1.1 is keg-only, which means it was not symlinked into /usr/local,
because macOS provides LibreSSL.

If you need to have openssl@1.1 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> ~/.zshrc

For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"

For pkg-config to find openssl@1.1 you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/openssl@1.1/lib/pkgconfig"

==> Summary
🍺  /usr/local/Cellar/openssl@1.1/1.1.1o: 8,089 files, 18.5MB
==> `brew cleanup` has not been run in the last 30 days, running now...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
Removing: /usr/local/Cellar/ca-certificates/2021-10-26... (3 files, 208.5KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/ca-certificates--2021-10-26... (117.6KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/fontconfig--2.13.1... (1.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/freetype--2.11.0... (957.0KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/gdbm--1.22... (256.5KB)
Removing: /usr/local/Cellar/openjdk/17.0.1_1... (639 files, 305.2MB)
Removing: /usr/local/Cellar/openjdk/18.0.1... (641 files, 307.7MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/openjdk--18.0.1.monterey.bottle.tar.gz... (181.3MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/openjdk--17.0.1_1... (180.2MB)
Removing: /usr/local/Cellar/openssl@1.1/1.1.1l_1... (8,073 files, 18.5MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/python@3.9--3.9.9... (13.8MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/sqlite--3.37.0... (2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/librsvg--2.50.7... (37.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libtool--2.4.6_4... (1004.8KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/netpbm--10.86.26... (1.8MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/xorgproto--2021.5... (695.5KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/jasper--2.0.33... (396.4KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/pango--1.50.0... (815.3KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/jpeg--9d... (311.5KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/glib--2.70.2... (6.3MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/gdk-pixbuf--2.42.6... (782.8KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/harfbuzz--3.1.2... (2.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/graphviz--2.49.3... (3.7MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libtiff--4.3.0.monterey.bottle.tar.gz... (1.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/icu4c--69.1... (27.2MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libx11--1.7.2... (2.1MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/gobject-introspection--1.70.0_2... (1.8MB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/libxcb--1.14_1... (919.9KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/webp--1.2.1_1... (888.6KB)
Removing: /Users/zoubaitao/Library/Caches/Homebrew/fribidi--1.0.11... (94KB)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/libpng... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/freetype... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/openjdk@11... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/maven... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/fontconfig... (49.6KB)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/gradle... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/openjdk... (64B)
Removing: /Users/zoubaitao/Library/Logs/Homebrew/maven@3.5... (64B)
Pruned 0 symbolic links and 2 directories from /usr/local
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew install kafka
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/kafka-3.2.0.monterey.bottle.tar.gz
Already downloaded: /Users/zoubaitao/Library/Caches/Homebrew/downloads/c03f1e5a559c1c99f9082da89614f93f10db9e72b4d9b568478e2ec94a952cdf--kafka-3.2.0.monterey.bottle.tar.gz
==> Pouring kafka-3.2.0.monterey.bottle.tar.gz
==> Caveats
To restart kafka after an upgrade:
  brew services restart kafka
Or, if you don't want/need a background service you can just run:
  /usr/local/opt/kafka/bin/kafka-server-start /usr/local/etc/kafka/server.properties
==> Summary
🍺  /usr/local/Cellar/kafka/3.2.0: 200 files, 99.4MB
==> Running `brew cleanup kafka`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew service restart kafka
Error: Unknown command: service
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services restart kafka
==> Successfully started `kafka` (label: homebrew.mxcl.kafka)
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start zookeeper
==> Successfully started `zookeeper` (label: homebrew.mxcl.zookeeper)
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start kafka
Bootstrap failed: 5: Input/output error
Try re-running the command as root for richer errors.
Error: Failure while executing; `/bin/launchctl bootstrap gui/501 /Users/zoubaitao/Library/LaunchAgents/homebrew.mxcl.kafka.plist` exited with 5.
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop kafka
Stopping `kafka`... (might take a while)
==> Successfully stopped `kafka` (label: homebrew.mxcl.kafka)
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop zookeeper
Stopping `zookeeper`... (might take a while)
==> Successfully stopped `zookeeper` (label: homebrew.mxcl.zookeeper)
```
## 二、Kafka集群
### 2.1、配置Kafka集群
配置Kafka集群，我们先进入到Kafka的安装目录，然后进入到配置文件目录

```bash
cp server.properties server1.properties
cp server.properties server2.properties
```

然后修改server*.properties文件

edit |server.properties |server1.properties |server2.properties |
|------|------|------|------|
|broker_id|broker_id=0|broker.id=1| broker_id=2|
|listeners|listeners=PLAINTEXT://:9092 |listeners=PLAINTEXT://:9093|listeners=PLAINTEXT://:9094|
|logs|log.dirs=/usr/local/var/lib/kafka-logs-0|log.dirs=/usr/local/var/lib/kafka-logs-1|log.dirs=/usr/local/var/lib/kafka-logs-2|

### 2.2、启动Kafka集群

```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-server-start /usr/local/Cellar/kafka/3.2.0/libexec/config/server.properties
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-server-start /usr/local/Cellar/kafka/3.2.0/libexec/config/server.properties
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-server-start /usr/local/Cellar/kafka/3.2.0/libexec/config/server.properties
```

### 2.3、创建topic
#### 2.3.1 创建topic
这里顺便说一句，kafka已经不再使用zookeeper保存topic的信息，而是采用broker来保存对应的信息，因此这里下面有--bootstrap-server localhost:9092而不是--zookeeper localhost:2181
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 1 --topic test
```
这条命令中的--replication-factor 3和--partition 1分别是什么意思呢？
* replication-factor表示将任意一个分区复制到多个broker上，也就是说relication-factor指定的数值一定要比我们启动的broker要小，否则就会报异常。下面这个实例就说明了这个问题。
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 4 --partitions 4 --topic broker_test
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Error while executing topic command : Replication factor: 4 larger than available brokers: 3.
[2022-07-09 15:43:55,669] ERROR org.apache.kafka.common.errors.InvalidReplicationFactorException: Replication factor: 4 larger than available brokers: 3.
 (kafka.admin.TopicCommand$)
```
* partition 3表示创建3个分区，我们开启了三个分区，如果我们的broker数量也是3的话，那么一个broker上面就会包含一个partition，如果一个partition的数量大于broker的数量时，就会通过hash的方式将剩下的partition存储到hash(partition) % brokerSize 后对应的broker。


  

#### 2.3.2、查看topic信息
查看所有已经创建的Kafka topic名称
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-topics --bootstrap-server localhost:9092 --list test
```
查看某一个topic的详情
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-topics --bootstrap-server localhost:9092 --topic test --describe
Topic: test	TopicId: Ccy0HpJEQAyH_a5XBdY7-A	PartitionCount: 1	ReplicationFactor: 3	Configs: segment.bytes=1073741824
	Topic: test	Partition: 0	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
```
#### 2.3.3、删除topic
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-topics --delete --bootstrap-server localhost:9092 --topic test
```
#### 2.3.4、创建consumer group

### 2.4、生产和消费消息
生产消息
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-console-producer --broker-list localhost:9092 --topic test
>Test Message1
>Test Message2
>Test Message3
>Test Message4     
```

消费消息
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning
Test Message1
Test Message2
Test Message3
Test Message4
^CProcessed a total of 4 messages
```
如果我想看每一个partition上有多少消息，应该怎么处理呢？

## 三、启停Kafka
启动zookeeper
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start zookeeper
==> Successfully started `zookeeper` (label: homebrew.mxcl.zookeeper)
```

启动Kafka
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services start kafka
==> Successfully started `kafka` (label: homebrew.mxcl.kafka)

```

 关停Kafka
 ```bash
 zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop kafka 
Stopping `kafka`... (might take a while)
==> Successfully stopped `kafka` (label: homebrew.mxcl.kafka)
 ```

关停zookeeper
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % brew services stop zookeeper
Stopping `zookeeper`... (might take a while)
==> Successfully stopped `zookeeper` (label: homebrew.mxcl.zookeeper)
```
## 四、Kafka重消费
首先看一下application.properties的相关配置，
```java
kafka.producer.test.bootstrap.servers=localhost:9092,localhost:9093,localhost:9094
kafka.consumer.test.bootstrap.servers=localhost:9092,localhost:9093,localhost:9094
kafka.consumer.test.session.timeout=15000
kafka.consumer.test.max.poll.interval.ms=10000
kafka.consumer.test.enable.auto.commit=false
kafka.consumer.test.batch.listener=false
```
这里配置的两次消费消息最大的间隔为10000ms，也就是第一次拉到的消息到第二次拉到的消息如果消费时间超出了10000ms那么就会提交失败。那么此时Kafka的broker就不会删除消息，下一次消费消息的时候就会重复拉取上一次未提交的消息。这里我们程序配置的max.poll.timeout.ms=10000ms，其实我消费程序中设置的睡眠时间为3000ms，那为什么还是会超出时间呢？


```bash
2022-07-10 12:22:36.273  INFO 3647 --- [ntainer#0-0-C-1] org.apache.kafka.clients.Metadata        : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Cluster ID: rB7bZWx2R06ebE66WjZdAQ
2022-07-10 12:22:36.274  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Discovered group coordinator localhost:9092 (id: 2147483647 rack: null)
2022-07-10 12:22:36.276  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] (Re-)joining group
2022-07-10 12:22:36.289  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Request joining group due to: need to re-join with the given member-id
2022-07-10 12:22:36.289  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] (Re-)joining group
2022-07-10 12:22:36.290  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Successfully joined group with generation Generation{generationId=43, memberId='consumer-testGroup-1-7c9240a8-21e2-4ef2-9331-2957ff563dad', protocol='range'}
2022-07-10 12:22:36.292  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Finished assignment for group at generation 43: {consumer-testGroup-1-7c9240a8-21e2-4ef2-9331-2957ff563dad=Assignment(partitions=[test-0, test-1, test-2, test-3])}
2022-07-10 12:22:36.299  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Successfully synced group in generation Generation{generationId=43, memberId='consumer-testGroup-1-7c9240a8-21e2-4ef2-9331-2957ff563dad', protocol='range'}
2022-07-10 12:22:36.299  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Notifying assignor about the new Assignment(partitions=[test-0, test-1, test-2, test-3])
2022-07-10 12:22:36.302  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Adding newly assigned partitions: test-1, test-0, test-3, test-2
2022-07-10 12:22:36.312  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-1 to the committed offset FetchPosition{offset=50, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9094 (id: 2 rack: null)], epoch=2}}
2022-07-10 12:22:36.312  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-0 to the committed offset FetchPosition{offset=0, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9092 (id: 0 rack: null)], epoch=2}}
2022-07-10 12:22:36.312  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-3 to the committed offset FetchPosition{offset=20, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9092 (id: 0 rack: null)], epoch=2}}
2022-07-10 12:22:36.312  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-2 to the committed offset FetchPosition{offset=20, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9093 (id: 1 rack: null)], epoch=4}}
2022-07-10 12:22:36.313  INFO 3647 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : testGroup: partitions assigned: [test-1, test-0, test-3, test-2]
2022-07-10 12:22:39.355  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"0"
2022-07-10 12:22:42.359  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"1"
2022-07-10 12:22:45.362  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"2"
2022-07-10 12:22:46.398  WARN 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] consumer poll timeout has expired. This means the time between subsequent calls to poll() was longer than the configured max.poll.interval.ms, which typically implies that the poll loop is spending too much time processing messages. You can address this either by increasing max.poll.interval.ms or by reducing the maximum size of batches returned in poll() with max.poll.records.
2022-07-10 12:22:46.398  INFO 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Member consumer-testGroup-1-7c9240a8-21e2-4ef2-9331-2957ff563dad sending LeaveGroup request to coordinator localhost:9092 (id: 2147483647 rack: null) due to consumer poll timeout has expired.
2022-07-10 12:22:46.399  INFO 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Resetting generation due to: consumer pro-actively leaving the group
2022-07-10 12:22:46.399  INFO 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Request joining group due to: consumer pro-actively leaving the group
2022-07-10 12:22:48.367  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"3"
2022-07-10 12:22:51.368  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"4"
2022-07-10 12:22:54.371  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"5"
2022-07-10 12:22:57.376  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"6"
2022-07-10 12:23:00.377  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"7"
2022-07-10 12:23:03.382  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"8"
2022-07-10 12:23:06.387  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"9"
2022-07-10 12:23:06.389  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Failing OffsetCommit request since the consumer is not part of an active group
2022-07-10 12:23:06.393 ERROR 3647 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : Consumer exception

java.lang.IllegalStateException: This error handler cannot process 'org.apache.kafka.clients.consumer.CommitFailedException's; no record information is available
	at org.springframework.kafka.listener.DefaultErrorHandler.handleOtherException(DefaultErrorHandler.java:155) ~[spring-kafka-2.8.2.jar:2.8.2]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.handleConsumerException(KafkaMessageListenerContainer.java:1762) [spring-kafka-2.8.2.jar:2.8.2]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1285) [spring-kafka-2.8.2.jar:2.8.2]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_301]
	at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_301]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_301]
Caused by: org.apache.kafka.clients.consumer.CommitFailedException: Offset commit cannot be completed since the consumer is not part of an active group for auto partition assignment; it is likely that the consumer was kicked out of the group.
	at org.apache.kafka.clients.consumer.internals.ConsumerCoordinator.sendOffsetCommitRequest(ConsumerCoordinator.java:1137) ~[kafka-clients-3.0.0.jar:na]
	at org.apache.kafka.clients.consumer.internals.ConsumerCoordinator.commitOffsetsSync(ConsumerCoordinator.java:1002) ~[kafka-clients-3.0.0.jar:na]
	at org.apache.kafka.clients.consumer.KafkaConsumer.commitSync(KafkaConsumer.java:1491) ~[kafka-clients-3.0.0.jar:na]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.doCommitSync(KafkaMessageListenerContainer.java:2978) [spring-kafka-2.8.2.jar:2.8.2]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.commitSync(KafkaMessageListenerContainer.java:2973) [spring-kafka-2.8.2.jar:2.8.2]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.commitIfNecessary(KafkaMessageListenerContainer.java:2959) [spring-kafka-2.8.2.jar:2.8.2]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.processCommits(KafkaMessageListenerContainer.java:2751) [spring-kafka-2.8.2.jar:2.8.2]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.pollAndInvoke(KafkaMessageListenerContainer.java:1313) [spring-kafka-2.8.2.jar:2.8.2]
	at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.run(KafkaMessageListenerContainer.java:1236) [spring-kafka-2.8.2.jar:2.8.2]
	... 3 common frames omitted

2022-07-10 12:23:06.393  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Giving away all assigned partitions as lost since generation has been reset,indicating that consumer is no longer part of the group
2022-07-10 12:23:06.393  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Lost previously assigned partitions test-1, test-0, test-3, test-2
2022-07-10 12:23:06.394  INFO 3647 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : testGroup: partitions lost: [test-1, test-0, test-3, test-2]
2022-07-10 12:23:06.394  INFO 3647 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : testGroup: partitions revoked: [test-1, test-0, test-3, test-2]
2022-07-10 12:23:06.394  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] (Re-)joining group
2022-07-10 12:23:06.396  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Request joining group due to: need to re-join with the given member-id
2022-07-10 12:23:06.396  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] (Re-)joining group
2022-07-10 12:23:06.397  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Successfully joined group with generation Generation{generationId=45, memberId='consumer-testGroup-1-aa1cbc3c-7464-448b-8ef3-d056e16567cd', protocol='range'}
2022-07-10 12:23:06.398  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Finished assignment for group at generation 45: {consumer-testGroup-1-aa1cbc3c-7464-448b-8ef3-d056e16567cd=Assignment(partitions=[test-0, test-1, test-2, test-3])}
2022-07-10 12:23:06.399  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Successfully synced group in generation Generation{generationId=45, memberId='consumer-testGroup-1-aa1cbc3c-7464-448b-8ef3-d056e16567cd', protocol='range'}
2022-07-10 12:23:06.400  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Notifying assignor about the new Assignment(partitions=[test-0, test-1, test-2, test-3])
2022-07-10 12:23:06.400  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Adding newly assigned partitions: test-1, test-0, test-3, test-2
2022-07-10 12:23:06.401  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-1 to the committed offset FetchPosition{offset=50, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9094 (id: 2 rack: null)], epoch=2}}
2022-07-10 12:23:06.402  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-0 to the committed offset FetchPosition{offset=0, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9092 (id: 0 rack: null)], epoch=2}}
2022-07-10 12:23:06.402  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-3 to the committed offset FetchPosition{offset=20, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9092 (id: 0 rack: null)], epoch=2}}
2022-07-10 12:23:06.402  INFO 3647 --- [ntainer#0-0-C-1] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Setting offset for partition test-2 to the committed offset FetchPosition{offset=20, offsetEpoch=Optional.empty, currentLeader=LeaderAndEpoch{leader=Optional[localhost:9093 (id: 1 rack: null)], epoch=4}}
2022-07-10 12:23:06.402  INFO 3647 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : testGroup: partitions assigned: [test-1, test-0, test-3, test-2]
2022-07-10 12:23:09.410  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"0"
2022-07-10 12:23:12.414  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"1"
2022-07-10 12:23:15.419  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"2"
2022-07-10 12:23:16.476  WARN 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] consumer poll timeout has expired. This means the time between subsequent calls to poll() was longer than the configured max.poll.interval.ms, which typically implies that the poll loop is spending too much time processing messages. You can address this either by increasing max.poll.interval.ms or by reducing the maximum size of batches returned in poll() with max.poll.records.
2022-07-10 12:23:16.476  INFO 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Member consumer-testGroup-1-aa1cbc3c-7464-448b-8ef3-d056e16567cd sending LeaveGroup request to coordinator localhost:9092 (id: 2147483647 rack: null) due to consumer poll timeout has expired.
2022-07-10 12:23:16.478  INFO 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Resetting generation due to: consumer pro-actively leaving the group
2022-07-10 12:23:16.478  INFO 3647 --- [ead | testGroup] o.a.k.c.c.internals.ConsumerCoordinator  : [Consumer clientId=consumer-testGroup-1, groupId=testGroup] Request joining group due to: consumer pro-actively leaving the group
2022-07-10 12:23:18.422  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"3"
2022-07-10 12:23:21.427  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"4"
2022-07-10 12:23:24.432  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"5"
2022-07-10 12:23:27.435  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"6"
2022-07-10 12:23:30.436  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"7"
2022-07-10 12:23:33.441  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"8"
2022-07-10 12:23:36.443  INFO 3647 --- [ntainer#0-0-C-1] c.z.c.s.impl.AlgorithmConsumerImpl       : consumer topic:test, message:"9"
```

## 原生API-KafkaProducer
1、这里使用原生的API来写数据，只设置Kafka集群地址，以及对应的序列化相关的数据
```java
@Slf4j
public class KafkaProducerMain {

    public static void main(String[] args) {
        test1();
    }

    private static void test1() {
        Map<String, Object> properties = new HashMap<>();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092,localhost:9093,localhost:9094");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);
        String topic = "test";
        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topic, String.valueOf(i), String.valueOf(i));
            kafkaProducer.send(producerRecord, new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    log.info("the recordMetadata::{}", recordMetadata.offset());
                }
            });
        }
        try {
            Thread.sleep(1000);
        } catch (Exception ex) {
            log.error("thread sleep error,", ex);
        }

    }
}
```

|配置项|解释|参考值|
|:---:|:---:|:---:|
|buffer.memory|||
|linger.ms|||
|batch.size|||
|acks|||
|retries|||
|max.block.ms|||


## 原生API-KafkaConsumer
## 重平衡-Rebalance

## 参考文档
[mac环境下使用brew安装Kafka(详细过程)](https://cloud.tencent.com/developer/article/1780636)

[macOS上使用brew安装Kafka](https://qizhanming.com/blog/2021/01/05/how-to-install-kafka-on-macos-via-brew)

[Kafka源码深度解析－序列7 －Consumer －coordinator协议与heartbeat实现原理](https://blog.csdn.net/chunlongyu/article/details/52791874)

[kafka系列七、kafka核心配置](https://www.cnblogs.com/wangzhuxing/p/10111831.html)

[Kafka生产者ack机制剖析](https://jiamaoxiang.top/2020/07/05/Kafka%E7%94%9F%E4%BA%A7%E8%80%85ack%E6%9C%BA%E5%88%B6%E5%89%96%E6%9E%90/)