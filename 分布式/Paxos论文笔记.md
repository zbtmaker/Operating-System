# Paxos论文笔记

当我在看Paxos论文的时候，我们知道Paxos论文的时候知道Paxos是一个一致性算法，那么为什么需要一致性算法呢。是因为随着一个网站或者一个应用的用户的增长，因此从单机到分布式，分布式对于用户而言应该是无感知的，他从应用维度只要看到的数据一致就可以了。

当用户写数据之后就又能够读取到数据。如何以一个公司的角度切入到一致性算法，为什么需要一致性算法。我们首先需要来回答为什么我们需要一致性算法。在这个背景下， 如果没有一致性算法会出现什么问题，有了一致性算法这些问题是否得到了解决。

在看视频提到一致性问题的时候总是说会存在多个proposer提交proposal，因为我们在现实开发过程中往往都是一个用户使用自己的记录，第一没有高并发。但是会存在多个用户修改同一个记录的问题，比方说我们的已下载次数。如果多个用户同时下载同一个用户，那么两个用户同时自增下载记录，如果无法在分布式系统中达成共识，那么就会出现这个应用在两次下载只修改一次的问题。这个例子似乎还是没有办法引入一致性算法的问题。

一个是互联网架构问题，
那么一致性算法需要解决两个问题：

一个是选举问题：选举问题就是在Leader宕机的情况下，能够尽快选择出一个Leader响应用户的请求。

另一个问题就是，Leader宕机之后，用户之前写的数据不会丢失，比方说用户之前写了X = X + 1，那么这个时候当Leader宕机之后，重新选举出来的Leader能够响应用户的请求，比方说用户这个时候用户在响应用户的读请求的时候，看到的X的值应该是2，而不是一个初始值1。一个是让请求方无感知（或者换句话说就是好像这个系统只有一个副本）

我现在陷入到一个看Paxos made simple这个理论进入到一个牛角尖了。其实我们可以先理解Paxos的single-paxos，然后看multi-paxos。然后进入到Raft论文中。



问题
* Paxos的构成是什么？Proposer、Acceptor、Learner又是做什么的。
* 问题：文章中提到Paxos会引起活锁，为什么呢？因为在Paxos中不存在Leader，一个方式就是Bully Algorithm算法选举一个Leader，然后这个Leader负责接受Client请求，这样就保证了只有一个ProProposer。
  * 首先需要了解什么是活锁，然后是什么原因导致的，
  * 然后看一下Bully Algorithm是如何实现选举的
  * Bully Algorithm选举之后是如何解决活锁问题的。

* 什么是Paxos
* 什么事singe-value
* 什么是multi-paxos
* Paxos是如何管理Proposer、Acceptor、Learner的退出和加入的。
  
## TODO List

* 看完Paxos相关视频；
* 完成学习资料中的页面和视频学习。
## Paxos made simple- PPT笔记
### 问题
* 在PPT当中提到2PC能够实现分布式系统达成一致的问题，那么2PC不能解决什么问题呢？
* 既然提到了上面的问题，那么2PC的理论或者说实现过程是什么样的？是如何解决分布式系统一致性问题。在解决分布式问题是在什么条件下？为什么脱离这个环境就无法实现一致性问题
* 在2PC算法基础上，人们提出了3PC方案来解决分布式系统一致性问题，但是3PC在一定情况下，会产生不正确的结果？我们需要了解3PC解决分布式系统问题，3PC是在什么背景下解决分布式一致性问题，在corner case无法满足的条件下是如何进一步提出解决方案的。
* Paxos主要解决两个问题，第一个问题是领导选举，另一个问题是主从一致
* Paxos中的Server的角色：propers、acceptors、learners。
* Paxos中的Server可能会失败，但是会重启也就是failure- recovery，集群中的机器可能会宕机，但是不会恶意伪造错误的信息误导其他的Server。
* Safety Properties：
  * only a value that has been proposed may accepted.
  * only a single value is chosen.
  * an agent never learns that a value has been chosen unless i actually has been.
* 其实我这里不理解什么是single value，
* 算法流程
  * propers选择一个提议号$n$，然后发送一个request给acceptors。
  * 如果acceptor还没有接受一个比这个提议号$n$更高的proposal number，因此acceptor就会接受proposal $n$。
  * proposer接受到大多数acceptors的回应，此时proposer会给acceptor发送一个accept message。
  * An acceptor that receives an accept message accepts the proposal unless it has responded to a prepare with a number greater than n。这句话是表示说如果一个收到一个accept message的acceptor要接受这个proposal除非这个acceptor已经回应一个准备好的number，这个number一定是大于n的。
* 从这个文件中可以看到，这是一个2PC的过程，首先是proposer提一个number，如果大多数acceptor接受了这个number，然后proposer会把这个single value发送给acceptor，文中没有详细信息说明在acceptor这个message的时候是否需要majority acceptor的同意呢？如果在这个阶段没有得到大多数acceptor的同意，那么此时proposer能够得到什么呢？文中似乎没有提到这个事情。

* 从这个文档其实可以看出来Paxos首先需要提出一个proposal number，如果acceptor中有超过一半的acceptor接受了这个proposal number，那么此时proposer就会把自己的value发送给acceptor。有个疑问，如果在第一步也就是acceptor在接受了proposer提出的proposal number也写入了自己的寄存器，但是就是在回复proposal的时候宕机了，然后proposer因为这一个acceptor的宕机，导致proposer无法获取超过半数的acceptor，此时这个法案就没有办法继续下去，那么Paxos应该如何处理呢。（今天先写到这里，实在是太困了，希望明天能够好好看完相关的论文，实在是太累了）
## Paxos made simple- 论文笔记
## Paxos made moderaely complex论文笔记
其实在读这篇论文的时候，没有搞清楚replica是一个什么样的角色，以为之前在看Paxos的时候会说存在三种角色，一个是Proposer、Acceptor、Learner。但是这篇文章只给出了一个replica的概念，就很模糊。

一些术语

* state machine：state machine由一个state集合，state的状态转换集合、当前state组成。state A转换到state A是允许的（如果用户的request是只读的command）。deterministic state machine概念，state的转换只是与operation和之前的state相关。其实这里的state machine可以理解为每一个operator对应一个state，多个operation对应多个state因此就构成了一个state collection。同时每一个operator都会导致state的transition（也就是状态的转换），从一个状态转换成另一个状态。
* asynchronous  network：这里的异步网络和我们之前在操作系统的同步和异步是不一样的/这是network中的一个专有名词，表示两台server在通信期间的时长是任意长的（因为在ATM网络中是专门划分带宽来实现网络通信的，因此通信的时长是固定的一个人说话，另一个人在很短时间就能接收到，具体关于asynchronous network可以去看《数据密集型应用设计》）
* crash failure：
* state machine replication：我们有一些state machine的副本，这些副本执行operation的顺序以及最终输出的结果和master state machine保持一致。

* 这里其实在文章中提到了一个stub routine.关于stub routine的概念参考[stub routine](https://stackoverflow.com/questions/4029313/what-is-a-stub-routine)。文中说客户端进程通过调用存根库通过网络发送command给Server然后等待结果。
* 而远端的stub routine是通过个
* 
关于一些参数的解读
* $k$：client
* <$k$, $cid$, $op$>：command， $cid$是client的唯一标识符，$op$ 表示client需要执行的操作。这是一个三元组，用一个三元组表示用户的请求。

* <$cid$, $result$>：replicas的对client的响应结果，$cid$表示client的唯一标识符，$result$标识replicas对client的响应。

* $slot$：表示replicas用于存储command（对应一个<$k$, $cid$, $op$>三元组），replicas用一个slots集合（也可以说是一个slot数组存储command）。

* ($s$, $c$)：标识slot $c$，存储的是proposal $c$。
* 一个replicas维护下面四个变量
  * $\rho.state$：应用程序的副本状态。

  * $\rho.slot\_num$：replicas当前的slot number。

  * $\rho.proposals$：副本过去提出的一组提案。

  * $\rho.decisions$：已经决定了的proposals集合。

关于一些不变量的解读

* 文中描述，为了避免不一致性，一个replicas会等待slot确定下来之后再升级自己的state和计算结果返回给用户之前，其实这里的逻辑和Raft的逻辑很相似了。
* Paxos并不要去replicas每时每刻都一致，但是需要replicas在将operation应用到application state的是时候是以相同的顺序。
## Paxos made live: an engineering perspective 论文笔记

## 学习资料
[Understanding Paxos](https://people.cs.rutgers.edu/~pxk/417/notes/paxos.html)

[Paxos算法](https://zh.wikipedia.org/zh-hk/Paxos%E7%AE%97%E6%B3%95)

[Consensus Protocols: Three-phase Commit](https://www.the-paper-trail.org/post/2008-11-29-consensus-protocols-three-phase-commit/)
[Paxos made simple-Conference](https://www.cs.columbia.edu/~du/ds/assets/papers/paxos-simple.pdf)
[Paxos made simple-PPT](https://15799.courses.cs.cmu.edu/fall2013/static/slides/paxos_made_simple.pdf)

[Paxos made live: an engineering perspective](https://news.cs.nyu.edu/~jinyang/ds-reading/paxos-live.pdf)

[Paxos made moderaely complex](https://www.cs.cornell.edu/home/rvr/Paxos/paxos.pdf)

