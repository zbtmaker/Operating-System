# MapReduce论文笔记

# 是什么
MapReduce 是Google用于处理文件系统的一个框架。MapReduce的目的是使软件工程师能够在没有并行分布式系统的经历也能够轻易的利用一个大规模分布式系统文件。

我们可以看到Google发表了很多的影响力很高的论文，是Google在做全球业务，不管是用户量还是数据量都是暴涨的。正式因为这样，需要解决的问题就很多。今天公司没有事情做，或者说没有新的东西，正是因为我们欧洲市场和其他市场没有什么起色，导致大家都没有什么事情可以做。

map：将输入数据

reduce：reduce从英文翻译成中文的意思是归约，其实也可以理解成将各个零散的数据归总起来。如果这么理解，那么和map又有什么区别呢？


## 目标
就是要屏蔽 **并行**、**容错**、**分布式数据**、**负载均衡**。因为是一个分布式系统，因此上面这几个因素都是需要仔细考虑的。

### 并行（parallelization）
就是将当前的任务分配给不同的机器，如果是单机，那么就是将任务分配给不同的线程。
### 容错（fault- tolerance）
什么是容错呢？
### 分布式数据（data distribution）
### 负载均衡（load balance）



# 怎么做

![这是图片](/img/mapreduce.png "Magic Gardens")
- 在这些机器中，其中一台机器比较特殊，这台机器就是master。集群中的机器分为master和worker。Master主要的工作就是将M个map任务和R个reduce任务分配给空闲的workers。所以在集群中的worker角色的机器既要执行map类型任务，又要执行reduce类型任务。
- 如果一个worker类型的机器被安排了map类型的任务，map需要读取对应被分割的文件。它从输入数据中解析键/值对，并将每对传递给用户定义的Map函数。由*Map*函数输出的\<key, value\>保存在执行map任务的worker内存中。
- \<key, value\>一开始保存在worker的内存中，然后以一定的周期将<key, value>写入到本地磁盘（这里说的本地磁盘是什么呢？是worker本身还是要写入磁盘中）。

- 这里如果执行map的worker中途宕机了，没有将内存中的\<key, value\>写入到local disk，作为master类型的worker需要做什么呢？master类型的worker是如何判定执行map或者reduce类型的worker是否宕机的。其实执行 map 函数或者 reduce 函数的worker类型的机器没有将master分配任务写入到 local disk，执行 reduce 函数的  worker 没有将文件输出到 output 文件，此时就这个任务就需要重新执行。

- 上面的描述其实会衍生出另外一个问题，就是当执行 map 函数的 worker 在将本地的键值对写入到 local disk 时，写了1/3的数据了，此时 worker 宕机了。此时 master 应该要分配一个空闲的 worker 来执行失败的任务。但是在已经写入了一半的 local disk 这些数据怎么处理，这里假设的是最新分配的worker要写入的 local disk 和之前的worker写入的 local disk是同一个。
- 在执行了 map 函数的 worker 如果已经执行成功了，worker 在向 master汇报的时候又是告知 master 自己写入数据的方式呢？master 在分配任务给新的 reduce 任务给新的 worker 时候应该告诉了 worker 之前已经完成 map 任务机器的写入了 local disk的起始位置和结束位置。
- 上面是假设把数据写入了 local disk，如果执行 map 函数没有将统计完成的键值对写入 local disk，只是把数据保存在内存当中，然后worker向master汇报自己已经完成了任务。也需要考虑已经执行完成的worker在和被分配了 reduce 人物的 worker之间的通信问题。

# 待改善
