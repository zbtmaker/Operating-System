
<div align='center'><font size = '5'>Zookeeper论文笔记</font></div></center>


<!-- TOC -->

- [一、Zookeeper数据结构](#%E4%B8%80zookeeper%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
- [二、Zookeeper读写](#%E4%BA%8Czookeeper%E8%AF%BB%E5%86%99)
- [三、Zookeeper Atomic Broadcast](#%E4%B8%89zookeeper-atomic-broadcast)
- [四、Zookeeper日志持久化](#%E5%9B%9Bzookeeper%E6%97%A5%E5%BF%97%E6%8C%81%E4%B9%85%E5%8C%96)
- [五、Zookeeper案例](#%E4%BA%94zookeeper%E6%A1%88%E4%BE%8B)
- [六、参考文档](#%E5%85%AD%E5%8F%82%E8%80%83%E6%96%87%E6%A1%A3)

<!-- /TOC -->






## 一、Zookeeper数据结构

## 二、Zookeeper读写
Zookeeper的机器分为Follower和leader两种角色，一个集群中只有一个leader，其他的都是follower。Follower会采用不同的策略处理Client的读请求和写请求。每个Follower在接受到Client请求时，会把写请求转发给leader，而针对读请求，会将本地结果返回给client。Follower处理读请求，Leader处理写请求，从这种多机读，一机写，写请求必然会有瓶颈。虽然多台机器可以处理读请求，提升了系统的读请求的TPS，但是随之带来的也会有一个问题就是读请求读到的是Follower的本地的state（state就是描述变量执行一连串的写请求后的结果，所以Zookeeper是一个写强一致性，但是读不是。

这里有一个疑问就是，现在有一个client，我们用$c_1$表示，发起了一个write request，我们用$r_1$表示，因为$r_1$这个写请求由Leader处理，然后Leader向集群中所有的Follower发起投票，但是因为有些机器比较慢，有些机器比较快。我们假设集群中有5台Server，其中Server1是Leader角色，其他的四台机器为Follower，我们用Follower2～Follower5表示。现在Leader1接收了$c_1$的请求参数，然后获的Follower2~Follower3的投票，也就是超过了半数的投票，所以这个时候就可以Leader就可以将请求$r_1$交给state machine执行，然后将结果返回给client。

如果这个时候请求$r_1$的请求因为Follower4和Follower5因为宕机了，或者因为网络问题，Follower4和Follower5的没有收到$r_1$，但是此时有client2向Follower4发起了读请求$r_2$，client3向Follower5发起了读请求$r_3$，因为之前的宕机，Follower4和Follower5并没有执行$r_1$的请求，所以此时client2和client3读取的只是过期的数据。其实还有好多环节都会出现这种读请求会读不到最新的数据，所以Zookeeper并不适用读强一直性的场景。所以Zookeeper是读强一致性，写非一致性，也就是<strong><em>Zookeeper不是一个强一致性的算法</strong></em>。

## 三、Zookeeper Atomic Broadcast

Zookeeper的领导选举和原子广播一致性是通过Zab这个协议来实现的。Raft基本上继承了Paxos的基本思想，为了解决死锁问题，采用单一Leader接受请求方式解决问题。Zookeeper的模式和Raft模式有些类似，也是采用一个Leader接受所有的请求（只有写请求，因为Zookeeper不要求读请求一致）。但是Raft在一个领导失效的情况下，会根据某一个Server的日志长度来选择Server是否能够成为一个Leader的其中一个条件，另外一个条件就是序列号一定要是最大的。

我们来看一下Zab这个协议是如何实现

* Zookeeper中各个服务器的角色，一次只有一个leader，其余的Server都充当Follower角色。所有的Server都可以接受请求，其中Follower只处理read request，如果client向Follower发起write request，Follower需要将write request转发给Leader处理。

* 针对writer request，Leader会首先将请求转发给所有的Follower，Follower收到了Leader的请求后，然后对本次请求进行投票，如果Leader收到了半数以上的Follower的赞同票，此时Leader再向Follower发起Commit请求。因为是一个二阶段的交互方式，如果一阶段成功，二阶段失败了，一阶段用户，二阶段交互的时候有一部分Follower并没有执行Commit操作，少部分用户执行完成Commit操作。如果此时Leader宕机了，那么是以已经执行完Commit作为标准呢？还是使用一阶段完成投票作为标准。

* zxid：是一个64bit的数字，其中低32位用于counter，高32位用于epoch。epoch主要用于选举，如果一个集群选举了一个新的leader，把epoch自增的同时，把低32位的counter置0，如下
   $$
   00000000 00000000 00000000 00000001 \ |\ 00000000 00000000 00000000 00000001
   $$
   正如上面展示的，在一开始epoch = 1，此时counter = 1，表示：。如果此时的leader宕机了，那么就需要将epoch自增1，然后将counter清零，如下所示：
   $$
    00000000 00000000 00000000 00000010 \ |\ 00000000 00000000 00000000 00000000
   $$

* 当leader宕机之后，是不是之前所有的Follower都可以参与竞争成为leader呢？我们可以考虑这样一种场景，5台机器，其中Server1、Server2、Server3、Server4、Server5。其中Server1作为leader宕机，其他的机器为Follower，此时Server2-Server5参与竞争，我们假设Server1在宕机之前接受了一个请求，假设只有Server2-Server3接受到请求并发出了投票，同时Leader执行了Commit，也只有Server2完成了Commit操作，Server3并没有执行Commit操作（相当于buff叠满了）。Leader宕机，所有的Server2-Server5都参与进来选举，如果此时Zab协议对选举不加任何的约束条件，Server2-Server5任意一台机器都可以成为Leader。

  我们假设Server4成为了Leader，此时同一个Client发出请求，Client的请求恰好是Server4服务的请求，我们假设Client在前一个Leader未宕机之前执行的操作是$x = x + 1$，其中$x$默认值为1，那么前一个Leader宕机之前返回给用户的值为$x = 2$。但是因为Server4和Server5并没有收到之前Leader的投票，也没有执行commit操作。所以此时Server4维持的状态依旧是$x = 1$。但是client在前一轮操作中已经看到了$x = 2$。因为前一轮服务的Leader的宕机了，Client恰好请求到Server4的读请求，此时Client拿到的结果就是$x = 1$。时间无法倒流，怎么就和之前的结果不同了呢？这样一种不一致性不是Zab协议想要实现的。


Zab分为三个阶段：发现（Discovery）、同步（Synchronization）、广播（Broadcast）
* Phase1-发现（ Discovery）
  * Follower给预期的Leader发送它的选举承诺CEPOCH($f.p$)消息
  * Leader一旦收到半数以上的Follower的CEPOCH($e$)，Leader就会提出一个方案NEWEPOCH($e'$)并将这个方案发送给Follower。其中$e'$的值比Leader收到的所有的Follower的  $e$值都要大。
  * 一旦Follower收到Leader（预期中的Leader）发送的$NEWEPOCH(e')$，如果Follower本身的epoch值 $f.p < e'$，Follower就会将自身的epoch值替换为leader发送的epoch值， $f.p = e'$。Follower返回$ACK-E(f.a,h_f)$给Leader，其中ACK的内容为Follower当前的epoch值$f.a$以及Follower历史已经提交的事物$h_f$。
  * Leader一旦接受到Follower发送的确认信息$ACK-E(f.a,h_f)$，Leader会选择其中的一个Follower的历史已提交的事物作为自己的历史已提交事物。那么这个选择的依据是什么呢？用公式表达就是，$\forall f' \in Q$，$f'.a < f.a$或者$f'_a = f_a \wedge f'.zxid \le f.zxid$ 。用通俗的话来讲就是选择zxid最大的那个。因为zxid上面已经讲过了构成方式，epoch大的则zxid一定大，epoch相等的情况下，counter大的zxid大。

* Phase2-同步（Synchronization）
  * Leader向所有的Follower发送$NEWLEADER(e',I_e')$。
  * Follower收到Leader的消息$NEWLEADER(E',T)$。如果Follower的$f.p \neq e'$ ，就会执行开始一个新的iteration（这个iteration表示什么呢？是重新选举还是什么？）如果$f.p =e$,此时Follower会执行$f.a = e'$，同时 $\forall<v,z> \in I_{e'}$，Follower会接受$<e',<v,z>>$，同时Follower会将自己已提交的历史事物为Leader的已提交的事物，$h_f = T$。在完成这个操作以后，就会返回$ACK-NEWLEADER(e',I_{e'})$结果给Leader。
  * Leader一旦收到半数以上的Follower返回的$ACK-NEWLEADER(e',I_{e'})$，此时Leader会向所有的Follower发送$commit$消息，此时Leader就完成了同步操作。
  * Follower一旦收到Leader的commit消息，Follower就

* Phase3-广播（Broadcast）
  * Leader $l$ 将提案proposal以zxid递增的方式发送给所有的Follower。每一个Proposal $<e',<v,z>>$，$epoch(z) = e'$，此外$z$继承了之前在 epoch 值$e'$广播的的所有的zxid 值。
  * Leader一旦收到了半数以上Follower针对提案$<e',<v,z>>$的投票，Leader会给所有的Follower发出$COMMIT(e',<v,z>)$请求。
  * 如果Follower处在选举状态中，则Follower f会触发$ready(e')$。
  * Follower $f$ 接受Leader $l$发出的提案，同时讲提案写入到$h_f$当中。
  * Follower $f$一旦接受到Leader发送的$COMMIT(e',<v,z>)$信息就会通过触发$abdeliver(<v,z>)$操作来提交事物$<v,z>$，同时Follower要保证之前所有的事物都已经提交，之前的事物表示$\forall <v',z'> \in h_f$，$z' < z$。
  * Follower $f$一旦接受到Leader发送的$COMMIT(e',<v,z>)$就会通过触发$abdeliver(<v,z>)$操作来提交一个事物$<v,z>$
  * Leader $l$在Phase3一旦收到Follower的$CEPOCH(e)$，Leader $l$就会提出$NEWEPOCH$和$NEWLEADER(e',I_{e'}\circ\beta_{e'})$。
  * Leader $l$一旦收到了Follower $f$针对$NEWLEADER(e',I_{e'}\circ\beta_{e'})$的确认信息，Leader将会发送一条commit信息给Follower $f$。Leader $f$也会将Follower f添加到集合$Q$当中。

* Leading
  * Leader和Follower之间通过heartbeat来确定对方是否宕机，Leader如果在timeout时间范围内没有收到半数以上Follower的心跳检测，那么Leader就会宣布放弃在epoch的领导角色，然后Leader进入$ELECTION$状态。
  * 这里和Raft有些不同的是，如果Follower在timeout时间范围内没有收到Leader的心跳检测信息，就认为 Leader已经宕机，但是Zab协议是Leader如果在timeout范围内没有收到半数以上的Follower的心跳检测，那么Leader就会进行重新选举。
  
* Following
  * 如果一个Follower参与到选举当中，Leader的epoch值一定要比Follower的epoch值要大，否则Follower会拒绝 protective Leader的选举请求。
  * 如果Follower在规定的timeout时间范围内没有收到Leader的心跳检测，那么follower就会放弃当前的Leader，从而进入$ELECTION$状态，同时继续算法中Phase 1的环节。（这里说的Phase1环节其实我也有些不解，怎么说呢？因为Phase1是发现环节，但是第一步就是Follower给Leader发送CEPOCH请求，但是正是因为和之前的Leader的心跳检测出问题了，现在又重新进入Phase1阶段，同时又和之前Leader进入第一阶段，这样就会进入新的循环。）


这里有一点说如果follower delivers<v,z>，那么primary 一定已经broadcast <v,z>。


如果一些follower delivers <v,z>，同时另外一些follower f' delivers <v',z'>,那么 f' delivers <v,z> 或者 f deliver<v',z'>。

论文中的follower deliver是什么意思，我一直以为是Leader执行执行abcast操作，还有deliver操作，这里在前文说当一个Process通过调用$abdeliver(<v,z>)$方法来发送transaction<v,z>

## 四、Zookeeper日志持久化
因为Zookeeper需要考虑宕机重启之后需要恢复之前的状态，如果只是将数据写入到内存，一旦服务器宕机，那么之前的状态就会丢失，然后需要和Leader之间进行数据同步。如果Follower在整个集群刚启动的时宕机，此时通过Follower和Leader之间通过通信就可以完成Follower的状态恢复。但是如果假如Leader和Follower之间启动已经一个月了，但是Follower这个时候所有的日志和状态都保存在内存了，宕机之后，Follower如果要和Leader之间通过数据同步完成状态恢复，可能需要好几天的时间才能同步所有的数据，在这几天过程中Leader又接受了新的写请求，所以Follower又落后了。那么现在有一个问题就是如何将历史的状态写入到硬盘（disk）当中，一旦某一个Follower宕机，此时Follower和Leader之间不再需要通过日志同步的方式完成状态（state）的同步。

其实在表达这里的写请求写入到本地和硬盘中，是因为如果一旦一个集群所有的机器都宕机了，那么整个集群就会出现状态丢失，因此就必须将日志一部分写入到本地，然后最近的历史状态需要写入到硬盘当中。这里把Zookeeper的日志持久化和Raft算法的日志持久化进行比较。

Zookeeper的日志持久化和Raft的方式大致类似，但是





## 五、Zookeeper案例

## 六、参考文档
[ZooKeeper: Wait-free coordination for Internet-scale systems](https://www.usenix.org/legacy/event/atc10/tech/full_papers/Hunt.pdf)


[A simple totally ordered broadcast protocol](https://www.datadoghq.com/pdf/zab.totally-ordered-broadcast-protocol.2008.pdf)