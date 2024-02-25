<div align='center'><font size = '10'>Zookeeper atomic broadcast theory and practice</font></div></center>

# Zab 协议
此论文中将Zab 协议划分为四个阶段，与原来的三个阶段，划分为四个阶段：选举、发现、同步、广播。

## Phase0-选举
当一个Follower在和Leader之间的心跳检测在规定的时间没有收到Leader的响应，那么Follower此时就会进入选举状态。其实与其说选举是一个阶段倒不如说选举是一个状态，代表了Server在一个时刻的状态。
  * 这里在文中描述的第一个阶段就是leader election，而Server与Server之间的关系是peer的关系。一个Server如果投票给自己，那么自己就成为了自己的prospective leader，如果投票给其他的Server，那么这个Server就会成为自己的prospective leader。这里我们用 $S_1$ 、 $S_2$ 、 $S_3$ 表示集群中三个服务器，如果 $S_1$ 首先进入 $election$ 状态，$S1$ 会首先投自己一票，然后会和其他的Server建立选举连接，但是这里有个细节没有讲清楚，就是 $S2$ 和 $S3$ 怎么样判定这个Server是一个有效的选举信息。比方说 $S_2$ 是当前的leader，那么怎么样让 $S_3$ 认为 $S_2$ 已经不是一个合格的领导了呢？这里在论文中其实是没有表述的。
  * 我们上面引申出一个问题，就是一个已经处于同步状态的广播状态的Server依据什么样的标准来判接收到的建立Peer关系的请求是应该处理还是应该丢弃。如果是我来设计这个系统，那么应该怎么设计呢？**暂且忽略这个问题吧**。

## Phase1-发现
当一个follower确定了一个prospective leader之后，就会和这个leader进行通信。论文说这个阶段的目的是在大多数Server找到最新的已经接受（我想这里的accepted的意思是committed的意思，也就是说已经提交的事物）。
  * 在这个阶段首先是Follower会发送 $FOLLOWERINFO(e)$，然后Leader会返回 $NEWEPOCH(e_{max})$ 类型消息。 Follower在接收来自Leader的 $NEWEPOCH(e_{max})$ 处理完成之后就会发送 $ACKEPOCH$ 消息，Leader接收到Follower的 $ACKEPOCH$ 后选择最大的Follower的history本地化后会进入到下一个阶段。所以这里的Phase1是在Phase0的基础上实现的，**而且Phase1最开始发起请求的还是Follower**。 
  * 这里从两个角度去看选发现过程，Leader会收到来自其他Follower的 $FOLLOWERINFO(e)$ 消息，这个消息表示follower已经选定 $Leader\ L$ 作为 $Follower \ F$ 的prospective leader。 $Leader\ L$需要做什么呢？从 $FOLLOWERINFO(e)$ 选择一个最大的epoch $e$作为新的选举的序号表示进入一个新的领导过程，这样做有什么好处呢？其实就是为了防止整个系统在同一个时间出现两个Leader，然后整个系统无法达成共识。
  *  $Leader \ L$ 通过收集所有 Follower 的epoch $e$ 值，选出epoch最大值 $e_{max}$ ，然后将这个最大值通过 $NEWEPOCH(e_{max})$ 类型消息返回给Follower。
  * 从任意一个 $Follower \ F$ 的角度来看，因为这个时候follower的状态其实处于发现状态的,$F.state = discovery$。如果prospective leader返回的$NEWEPOCH(e_{max})$消息中$e_{max} < e_{F}$也就是说这个prospective leader选出来的最大的epoch比follower的epoch值要小，那么就说明这个 prospective leader在选举过程中出现了网络或者其他因素，导致其选举落后了。此时 $Follower \ F$ 就需要重新进入选举状态，也就是说 $F.state = election$ 。那么这个时候follower是否需要发送一个响应结果给Leader呢？我想应该是不用的，这个时候如果采用的是一个懒惰的政策，等Leader的再一次请求的时候直接拒绝就好了，虽然这里Followed并没有想一次选出领导的方式。此时Follower就会会退到 $Phase 0$。
  * 如果 $Follower \ F$ 发现prospective leader返回的 $NEWEPOCH(e_{max})$ 消息中 $e_{max} > e_{F}$，此时就说明这个leader获取的半数以上的Server的投票， $Follower \ F$ 就需要接受这个prospective leader作为自己的Leader，同时 $Follower \ F$就需要存储一些变量， $F.acceptedEpoch = e_{max}$。此时Follower 和Leader之间就进入了下一个阶段，Follower和Leader之间也需要慢慢的去同步数据。此时 $Follower F$会发送信息 $ACKEPOCH(F.currentEpoch, F.history, F.lastZxid)$ 给 prospective leader。最后 $Follower F$ 会修改本地的状态，标志其进入到下一个状态。 $F.state = synchronization$。这里我们可以看到 $Follower F$ 在把自己的epoch值( $F.currentEpoch$ )，已经接受的transaction（$F.history$）、已经接受的最后的lastZxid都发送给 prospective leader。**此时 $Follower\ F$ 进入到下一个阶段$Phase2-synchronization$**
  * $Leader L$ 在收到所有的Follower的 $ACKEPOCH(f.currentEpoch,f.history, f.lastZxid)$ ，从所有的Follower中选择current Epoch最大的那个Follower的history，Leader会将这个history设置为自己的history。简单表述为 $\forall f` \in Q \setminus \{f\}$ ，就是要满足 $f`.currentEpoch < f.currentEpoch$ 或者是 $f`.currentEpoch = f.currentEpoch \land f`.lastZxid = f.lastZxid$ 。就是无论如何都要从 follower中选择接受了更多transaction的 $f.history$ 复制到Leader的 $L.history$ 。这么做的一个好处就是能够最大程度的恢复到前一个epoch的Leader已经提交的（committed）的所有transaction。执行这些操作后Leader也进入了 $Phase2$-同步。
  * 这里还有一个疑问就是如果Leader没有收到足够多的Follower的 $ACKEPOCH$ 请求会这么样呢？是不是Leader就无法进入到下一个阶段，那有没有一种可能就是Follower已经进入到next phase，但是Leader还处在current phase，如果存在这种情况，那么ZAB协议又是保证Leader和Follower同时进入到一个phase，然后完成next phase的数据呢。

## Phase2-同步
同步逻辑包含两个部分，一部分是Leader宕机之后的恢复，另外一个部分就是利用Leader的update功能将前一个epoch的leader已经提交的数据同步给其他所有的Follower使整个系统达成一致。
  * 在Phase1，一开始的请求有 $Follower \ F$ 发出具体的请求，最终由 $Follower \ F$ 先进入到Phase2，等Leader处理完Follower发送过来的history之后，此时的Leader为了让整个集群达成一个共识，之前由Leader收集Follower的最多已经提交的transaction，然后取最长的那个。现在Leader已经处理完了所有的history，为了让整个系统达成一致，必须把Leader的已经提交的history同步给半数以上的Follower才能实现整个系统的一致性。Leader会发送消息 $NEWLEADER(e_L, Leader.history)$ 给所有的Follower。
  *  $Follower \ F$ 收到Leader的 $NEWLEADER(e_L, Leader.history)$ ，此时如果Follower的epoch值 $e_F > e_L$ 此时表明之前的phase流程都已经过期了，也就是说 $Leader \ L$ 的epoch值已经过期了，此时 $Follower \ F$的状态就需要变更 $F.state = election$。
  * 如果 $Follower \ F$ 的epoch值 $e_F = e_L$，也就是表明Leader和Follower是在之前的Phase1中转换过来的。所以 Follower这个时候就需要将Leader的history进行对比，看看自己是不是提交了一些Leader没有的数据，或者是还有一些transaction没有提交。看论文中的表述是 $Follower \ F$ 会accept$Leader\ L$ 的所有的history都提交到本地（注意这里并不是直接commit，只是暂时存储在本地，具体的commit或者说deliver操作要等Leader收到半数以上的 $ACKNEWLEADER$ 响应后才会执行commit或者说是deliver操作）。 $Follower \ F$ 将所有的 $Leader \ L$的同步的事物同步到本地后，会返回一个 $ACKNEWLEADER(e,H)$ 给Leader。
  *  $Leader \ L$ 在收到半数以上的Follower的 $ACKNEWLEADER(e,H)$ 响应后，会给所有的Follower发送$COMMIT$请求。**此时 $Leader \ L$ 进入Phase3**。
  
  *  当 $Follower \ F$ 收到 $Leader\ L$ 消息后，此时就需要执行deliver操作将所有的事物交给process进行处理。**此时 $Follower \ F$ 进入Phase3**。

## Phase3-广播
  $Leader L$ 收到 $Follower \ F$的写请求（注意，对于Zab协议对于读请求都是由各自的Follower处理的，针对写请求，Follower会把请求转发到Leader处理）。在广播阶段主要处理Follower的写请求，同时如果有新的Follower申请加入Leader，那么Leader负责将历史请求同步给新加入的Follower。
  * 因为Follower不处理写请求，而是将请求转发给Leader。同时Zookeeper会保证如果一台Server没有宕机，那么一台机器如果之前的写请求是由Follower1处理的，那么在后续的读请求也会由 Follower1处理.这种方式虽然没有保证整个集群的线性一致性，但是保证了整个集群实现高吞吐量，这也是Zookeeper的优点。但是这里也有一个问题就是如果在写完之后，这台机器宕机的同时，Leader也宕机了，这个时候整个集群处于选举过程中，因为Zookeeper保证了没有Leader也是可以处理读请求的，那么会不会把这个请求转发到一个慢Server上处理，正因为这个慢Server每次响应Leader的请求都慢一些，所以Client之前的写请求，这台慢Server没有响应Leader的 $proposal<e',<v,z>>$ 请求，Client看到的就是旧数据。或者是说Leader没有宕机，但是响应这台机器的请求因为某种原因宕机了。那么这个Client的请求会不会被分配到慢Server上，Zookeeper又是如何避免这个问题的。
  * 首先来看写请求的处理流程。Follower收到Client的请求 $Proposal(e', \left \langle v,z \right \rangle)$ ，首先会将请求转发给  $Leader \ L$，Leader会首先将请求发送给各个Follower。
  * Follower收到 $Leader \ L$ 的请求后，会将 $Proposal(e', \left \langle v,z \right \rangle)$ 记录在 $F.history$中。 $Follower \ F$ 处理完成之后会发送 $ACK(e',\left \langle v,z \right \rangle)$ 请求给$Leader \ L$。
  * $Leader \ L$ 在收到半数以上的Follower的 $ACK(e',\left \langle v,z \right \rangle)$回复后，就会给各个Follower发送$COMMIT(e',\left \langle v,z \right \rangle)$ 请求
  * Follower在收到Leader的 $COMMIT(e',\left \langle v,z \right \rangle)$ 后会拿本地的Zxid和Leader的Zxid比较，如果 $z_f < z_l$ 此时就会先提交 $Follower \ F$ 之前的事物，等之前的事物提交之后，再处理当前的请求 $COMMIT(e',\left \langle v,z \right \rangle)$ 。如果当前 $Follower \ F$ 之前的事物已经处理完成了，就会立即提交当前事物给process处理。
  * Leader在BroadCast阶段另一件事情就是如果有新的Follower申请加入集群，那么Leader需要将自己的 $L.history$同步给new Follower。比方说，如果整个集群运行了三年，那么是需要new Follower将历史的数据全部同步嘛，还是说我只同步最新的一个状态就可以了，如果是要同步三年的历史数据，那么一个Follower可能在短期内是无法实现全部同步，应该是会有压缩的。我们可以看一下具体的信息。
  * 当一个新的Follower想要加入一个集群时，首先我们可以理解Follower应该是不知道哪台机器是Leader，那么集群中就可以定一个一个新的请求，比方说new Follower向集群中某一台机器发起了请求，请求命中了另一台机器也是Follower角色，此时当前的Follower就会把Leader信息返回给Leader，同时告诉new Follower这是一个重定向协议，你应该重新和Leader建立协议（这个逻辑有点像Redis的slot寻找的逻辑）。
  * 第一步建立起链接之后，Leader收到了来自new Leader 的 $FOLLOWERINFO(e)$ 的请求，此时Leader会发送两个响应给Follower，这里有点奇怪，为什么在实现过程时没有合并两个请求为一个请求发送。这里的两个响应一个是 $NEWEPOCH(e')$ 主要是同步当前的的epoch值给 new Follower。另一个就是 $NEWLEADER(e', L.history)$，主要是把Leader的已经提交的历史的日志同步给新Follower。
  * new Follower收到Leader发送的两个信息，并将Leader的历史日志保存到本地后，就会返回一个 $ACKNEWLEADER$ 响应给Leader，Leader此时就会发送一个 $COMMIT$ 给Follower，follower此时才会将自己的日志交给process去提交。到此一个new Follower算是加入了这个集群。

## Recovery Phase
在具体实现中将 $Phase1-discovery$ 和 $Phase2-synchronize$ 合并为一个阶段为Recovery Phase。具体实现流程为：

* $Follower \ F$，向prospective leader发送请求 $FOLLOWER(F.lastZxid)$给$Leader \ L$ 。
*  $Leader \ L$ 收到 $Follower \ F$ 的消息 $FOLLOWERINFO(f.lastZxid)$ 后就需要将自己的 $L.lastZxid$ 发送给 $Follower \ F$。如果 $f.lastZxid > L.history.lastCommittedZxid$ 那么$Leader L$ 会给 $Follower \ F$ 发送 $TRUNC(L.history.lastCommittedZxid)$。
* 如果 $f.lastZxid > L.history.lastCommittedZxid \wedge f.lastZxid  < L.history.oldThreshold$，此时 $Leader \ L$ 会给 $Follower \ F$发送 $SNAP$ 消息。
* 如果 $f.lastZxid > L.history.lastCommittedZxid \wedge f.lastZxid ≥ L.history.oldThreshold$，此时 $Leader \ L$ 会给 $Follower \ F$发送 $DIFF$ 消息。
* 如果 $Follower \F$，接收到 $Leader \ L$的 $TRUNC(lastCommittedZxid)$ 消息，$Follower \ F$ 会将 $L.lastCommittedZxid$ 到 $F.lastZxid$ 历史数据全部删除
* 如果 $Follower \F$，接收到 $Leader \ L$的 $DIFF(H)$ 消息，$Follower \F$ 会提交 $H$ 中所所有的事物。
* 如果 $Follower \F$，接收到 $Leader \ L$的 $SNAP$ 消息，$Follower \F$ 会将leader的snapshot全部存储到数据库中，然后提交存在变化的事物。



## Fast Leader Election
vote : 

id : 

state : 机器的状态有三种，一种是election、leading、following。如果Server的状态是election， 

round :

对于一个notification消息时四元组， $(vote, id, state, round)$，Peer之间是Server之间进行通信。

一台Server一开始把票投给了自己，发送notification $(vote, id, state, round)$给其他的Server。

如果发送的Peer不是在 $election$ 状态，那么当前Peer会更新自己的
当 Peer检测到大多数其他的peer达成了一个公共的投票，那么当前Peer返回自己的投票信息，然后决定自己成为一个leader或者是follower。


# Conclusion
：看完了整个Zab协议之后还是对以下存在疑问：

* leader具体是如何选择出来的，Peer建立之后的prospective leader到真正的leader之间的出现问题然后又是如何建立请求的。
* new Follower在复制的时候是复制Leader整个历史，还是说某一个时刻的最终态，然后将这个点之后的数据重新复制。
* 当Client先写成功，然后当前Follower宕机，Client被分配到另一台慢Server，那么Client无法读取到刚才写成功的数据，这里会出现歧义。

# Compare
和Raft之间的比较：
* Raft是一个强一致性的算法，不管是读还是写，都要求保持强一致性。Zab协议是一个写强制一致性，但是对于读并没有任何要求，要求的局部一致性。
* Zab协议因为不要求强一致性，所以读写可以分离，这样整个集群的读性能会随着集群机器的增加而增加。而Raft要求强一致性，所以增加机器反而会导致集群性能下降，因为读写都是由Leader来处理。
* 相比较Raft协议，Zab协议的论文读起来没有Raft协议那么清晰，Zab协议还是留了很多工业上的模糊点没有讲清楚。

