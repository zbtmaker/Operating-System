# MapReduce论文笔记

##  是什么
MapReduce 是Google用于处理文件系统的一个框架。MapReduce的目的是使软件工程师能够在没有并行分布式系统的经历也能够轻易的利用一个大规模分布式系统文件。

我们可以看到Google发表了很多的影响力很高的论文，是Google在做全球业务，不管是用户量还是数据量都是暴涨的。正式因为这样，需要解决的问题就很多。今天公司没有事情做，或者说没有新的东西，正是因为我们欧洲市场和其他市场没有什么起色，导致大家都没有什么事情可以做。

map：将输入数据

reduce：reduce从英文翻译成中文的意思是归约，其实也可以理解成将各个零散的数据归总起来。如果这么理解，那么和map又有什么区别呢？


## 目标
就是要屏蔽 **并行**、**容错**、**分布式数据**、**负载均衡**。因为是一个分布式系统，因此上面这几个因素都是需要仔细考虑的。

### 并行（parallelization）

就是将当前的任务分配给不同的机器，如果是单机，那么就是将任务分配给不同的线程。

### 容错（fault- tolerance）

什么是容错呢？一方面关注的是high availability，另外一方面关注的是high recoverability。其实就是要保证一个集群在任何时候都是可以用的，另外就是集群在遇到一部分机器宕机的时候，能够在其他机器处理问题时能够尽快恢复。其实high availiability 和high recoverability一个时机器能够作为备份处理，另外一个目的就是当运维将机器reboot（我理解的是重启）之后能够加入到cluster继续处理分布式任务。

这里我们可能会觉得很奇怪，为什么认为一台机器加入到cluster 是一个着重需要讨论的点。这是因为集群在执行任务后，会产生大量的数据，这种大量的数据造成新加入到集群的机器需要花费大量的时间来同步在宕机过程中产生的数据。同时哪些是新产生的数据，哪些又是需要被丢弃的数据，这些都是一个有效的机制来实现的。


### 分布式数据（data distribution）
### 负载均衡（load balance）



##  怎么做

![这是图片](/img/mapreduce.png "Magic Gardens")
### 流程
在这些机器中，其中一台机器比较特殊，这台机器就是 master。集群中的机器分为 master 和 worker。master 主要的工作就是将 M 个 map 任务和 R 个 reduce 任务分配给空闲的 worker。所以在集群中的worker角色的机器既要执行map类型任务，又要执行reduce类型任务。
* master 将分割好的文件将文件发送给 执行 map 函数的 worker类型的 server。
* worker 执行 map 函数后，首先数据写入到worker的本地内存，然后将数据写入到本次磁盘。
* worker 执行完成以后，会返回给 master 一个 OK的恢复，同时将键值对的文件地址返回给master。
* master 观察集群中空闲的 worker，然后分配这些 worker 执行 reduce 函数。注意 执行reduce 函数的 worker 一般都会从多个已经执行完成 map 函数的机器通过 rpc 的方式将 \<key, value\> 从 远程读取过来。
* 当一个reduce worker 接收到master的通知后，会通过rpc调用将 map worker存储的中间键值对全部读取到本地。当一个reduce worker读取到所有的中间值以后，会将键值对根据 key 进行排序。
* reduce worker排序好的key以及对应的中间值都提交给用户写好的reduce function，用户的Reduce 函数的输出结果都会写入到output file。
* 当所有的 map task和reduce task全部执行完成以后，master worker会唤醒用户程序，此时程序调用又回到用户程序调用Map Reduce函数处。

### master 数据结构

master需要存储每一个map task和reduce task的状态和唯一标识符（相当于一个ID），状态分为（idle-空闲，in-progress-处理中，completed-已经完成）。

master worker和 map worker和reduce worker之间需要进行通信。map worker完成map function后需要上报R个文件的位置和大小。这里的R是reduce worker的数量。其实map worker完成中间键值对统计之后不是存储在一个文件，而是将数据通过hash之后存储在R个文件中。为什么要存储在R个文件中，是因为考虑系统中还有R个reduce worker。

### worker failure
master 是如何知道 map 或者 reduce worker已经宕机了。，master定期发送 rpc 请求给 worker，如果 worker 在规定的时间没有恢复 master，那么 master 就判定 worker 就已经标记当前 worker状态为失败。如果一个 map task或者是 reduce task 运行在一个已经失败的worker上，那么master worker 会重新调度map task或者是 reduce task执行。


如果一个reduce task已经执行完成了就不需要重新执行了，因为reduce worker最终的数据都是写入到GFS（Google File System）。

如果一个 map task 首先被 worker A 执行，然后因为 worker A 失败了，此时 master worker 将这个 map task 重新安排到 worker B。master 会通知那些已经从 worker A 读取的 reduce worker，然后重新从 worker B读取数据。对于那些没有从 worker A读取数据的 reduce workers，可以直接从 worker B 读取数据。

已经完成的 map task 可以在一个failure的worker上重新执行，是因为执行map task的结果存储在 worker的本地磁盘，如果一个 worker 执行失败了，此时 master worker 和 reduce worker 都无法和 map worker通信，因此可以重新执行。但是文中没有说明新执行的数据结果存储在哪里（我想如果重新执行了，此时是可以重新开 R 个文件，然后将数据写入到这 R 个文件，然后）。

### master failure
如果master task死亡，一个新的机器可以从原来的机器的checkpoint状态读取数据。因为目前整个系统只有一台 master 机器，此时会放弃整个MapReduce计算。Client可以检查这些条件，然后重试MapReduce操作。

### Sematics in the Presents of Failures

一个map task完成以后，worker会发送一台哦消息给master，发送的消息中会包含worker的ID和R个文件的file name(这里可以认为这一次的file name是针对本次 map task 的R个文件，如果重新执行，又会重新生成的)。

如果一个reduce task在多个任务上执行，如果多个reduce worker同时执行完成，最终只有一个worker能够执行成功rename操作，这里说的是rename操作，并没有说写入的时候是怎么操作的，其实如果多个reduce task要往一个文件里面写数据，那么是不是表明多个reduce worker执行的结果是一致的。难道都是往一个文件里面写吗？还是说写入到多个文件。

## 问题
**问题1**：我们首先从 master 分配 map 任务给空闲 worker 来看。master 一次只会分配一个 map 任务给一个 worker。那么问题如下如果一个master 一次分配一个 map 任务给一个 worker，当 worker 执行完一个 worker 以后。worker 会向 master 回复自己已经完成的任务，同时将键值对存储的位置也汇报给 master。如果此时没有多个worker同时完成了 map 函数，那么此时又还有其他 map 类型任务需要执行，然后整个集群只有这一台机器是空闲的，那么 master 应该如何分配任务，是让 worker 继续执行新的 task 还是等待？

**问题2**：继续上一个**问题1**，如果一个 worker 类型的机器被安排了 map 类型的任务，map 需要读取对应被分割的文件。它从输入数据中解析键值对，并将每对传递给用户定义的 map 函数。由 *Map* 函数输出的 \<key, value\> 保存在执行 map 任务的 worker 内存中。\<key, value\> 一开始保存在 worker 的内存中，然后以一定的周期将 <key, value> 写入到本地磁

    这里说的本地磁盘是什么呢？是 worker 本身还是要写入磁盘中。这里说的 local disk 和我笔记本电脑中硬盘类似概念，也就是说这里的 local disk 其实不是基于一个分布式系统，只是服务器自己挂载的一个本地磁盘。

这里如果执行 map 的 worker 中途宕机了，没有将内存中的 \<key, value\> 写入到 local disk，作为 master 类型的 worker 需要做什么呢？master 类型的worker是如何判定执行 map 或者 reduce 类型的 worker 是否宕机的。其实执行 map 函数或者 reduce 函数的 worker 类型的机器没有将master分配任务写入到 local disk，执行 reduce 函数的  worker 没有将文件输出到 output 文件，此时就这个任务就需要重新执行。

**问题3** ：当执行 map 函数的 worker 在将本地的键值对写入到 local disk 时，写了1/3的数据了，此时 worker 宕机了。此时 master 应该要分配一个空闲的 worker 来执行失败的任务。但是在已经写入了一半的 local disk 这些数据怎么处理？

这里假设的是最新分配的 worker 要写入的 local disk 和之前的worker写入的 local disk是同一个。**这个问题在论文的 Fault Tolerance 会讲明白**。

在执行了 map 函数的 worker 如果已经执行成功了，worker 在向 master 汇报的时候又是告知 master 自己写入数据的方式呢？master 在分配任务给新的 reduce 任务给新的 worker 时候应该告诉了 worker 之前已经完成 map 任务机器的写入了 local disk 的起始位置和结束位置。

上面是假设把数据写入了 local disk，如果执行 map 函数没有将统计完成的键值对写入 local disk，只是把数据保存在内存当中，此时执行 map 函数的 worker 不会上报执行成功，map 函数执行成功的标志应该是 worker 将执行结果通过 Hash 的方式将结果均匀分散到 R 个文件写入到 local disk。所有的结果写入到 local disk 结束后，会将


执行 reduce 函数的 worker 将键值对读取到本地后，进行最后一步归总，这里是将所当前 worker 需要处理的所有数据归总完成后根据 key 依次写入到 output 文件，还是一次性将数据全部写入到 output 文件，如果是依次写入，那么此处也需要考虑执行 reduce 函数的 worker 中途宕机问题。如果 reduce worker 宕机，而且已经写入了一部分归总的数据到 output 文件，此时 master 经过一定的机制检测到 worker 宕机。master 重新分配一个空闲的 worker 重新执行 reduce 任务，但是 master 在执行之前，其实是不知道之前的 worker 写入了多少数据到 output 文件。
 
根据论文和视频中的说法是，执行 reduce 的 worker 会把之前已经执行完成 map 的 worker 的键值对 fetch 到本地之后才会执行 reduce 函数，所以这里需要考量的就是被读取的 worker 在读取过程中宕机之后 master 的检测问题，然后 reduce 和 master 之间的通信细节问题。不存在一致性问题。

根据上面描述的，如果是不是 master 只管那些已经完成任务的数据，对于那些中途宕机的任务的数据完全不管是不是就可以。不管是可以的，但是有一个问题就是针对执行一个任务的垃圾数据应该如何被清除掉呢？

其实这里关于 reduce 函数是怎么执行的，我好像没有一个很明确的方式，比方说我现在有1000个文件，然后这1000个文件交给了500个 worker 执行输出 500 个 local disk 本地文件，每个文件都存储了很多的键值对，此时 master在分配任务的时候是需要怎么做呢？这一块好像论文没有提及，还是说我没有仔细看。

    在论文中提及到当 map 函数执行完成以后，reduce 函数会执行多个 map 函数的输出结果，然后将多个结果的文件进行归总。那么在归总的过程中，reduce 函数的机器是如何知道需要对多少个 map 函数输出的结果进行归总的呢？这个具体的分配算法是如何确定？


在视频中讨论的是其实 MapReduce 框架其实不负责split file，这些都是user program 需要 map reduce执行的时候已经让 GFS（Google File System）处理好了。
