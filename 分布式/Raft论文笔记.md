# Raft论文笔记
<!-- TOC -->

- [Raft论文笔记](#raft论文笔记)
  - [一、选举 - Leader](#一选举---leader)
  - [二、复制 - Replicate](#二复制---replicate)
  - [三、安全 - Safety](#三安全---safety)
  - [四、集群成员变化](#四集群成员变化)
  - [五、Log压缩](#五log压缩)
  - [六、Client请求](#六client请求)
  - [TODO List](#todo-list)
  - [学习资料](#学习资料)

<!-- /TOC -->
Raft主要关注的三个方面，
* Leader选举
* log复制
* 安全问题
解决好上面三个问题，一个基本的分布式系统就可以做好了。
## 一、选举 - Leader
其实在阅读[Raft网站](https://raft.github.io/)的时候有一些疑问，这里主要从选举、复制（目前只是知道这两个方面）。

首先列举选举相关问题：
* 选举一问：因为Raft集群服务器存在三种角色，Follower、Leader、Candidate。Follower和Leader是选举之后的状态，那么当一个Leader宕机之后。各个Server的角色是什么样的呢？比方说之前的Follower角色，现在会变成什么样的角色呢？
  
  * 在Raft论文中，一个Server在一个Cluster中的最开始的角色时Follower角色，我们假定在一个选举完成的状态下，因为Leader会以一个固定的频率给集群中的Follower发送心跳检测，如果Follower在其设定的timeout超时时间之前还没有收到Leader的心跳检测，此时Follower就会进入Candidate角色；
  * Candidate角色开始向集群中其他的Server发送选举请求（这里的其他Server是包括Leader角色的Server），如果在一个选举周期内，没有投出一个有效的Leader（这里一个周期内，如何算是投出一个有效的Leader，我们后面做出说明），此时Candidate角色的Server的角色保持不变，进入下一轮投票；
  
  * 如果Candidate角色的Server在投票期间获得了Cluster中大部分的Server的投票，那么Candidate角色就会转变为Leader角色；如果Candidate在选举过程中收到了其他Server发送的AppendEntries RPC，此时Candidate判断自己的term和发送请求的Server的term进行比较，如果发送AppendEntries RPC的Server的term和自己一样大或者比自己的要大，那么Candidate状态会变更为Follower状态。
  
  * Follower角色又是如何发生变化的呢？这里说的是如果在election timeout没有收到Leader的心跳检测，那么会从Follower状态转变成为Candidate状态；或者是在election timeout没有收到Candidate的Request Vote的情况下也会转变为Candidate状态。存在当前term大家都没有获得半数以上的投票，此时会进入下一轮选举，那么在下一轮选举时， Server的状态时延续上一轮还是大家都进入Candidate状态，这个事情论文里面并没有详细讲。在没有后续Safety文章中这个逻辑显得无所谓，但是后面就需要阐明这个问题了。
  

* 选举二问：在上面我们提到一个选举周期，在一个选举周期呢？
  
  * Raft在论文中提到吧time分割成不同的term，每个term的时长是不一样的，在每一个term开始时都需要进行选举，选举完成之后开始响应Client的请求。如果自一个term中没有选举出合适的Leader（一个集群中超过一半的Server都投这个Server作为Leader），那么此时term就会结束，同时集群中的每一个Server的term在前一个term基础上自增。

  * 这里也提到，集群中的Server在通信时会传递自己的term，如果一个Server发现自己的term比其他Server的term要小，那么这个Server就需要升级自己的term。如果一个Server收到请求的term，这里在文中被称为stale term（过期term）， Server会拒绝请求。

  * 虽然我觉得这种方式不大靠谱，因为每个term都有选举，选举完成之后才能服务Client的请求，这个在现实的互联网世界，基本上时不可能的。那么实际的数据又是怎样的呢？，可以看一下 ETCD、Redis的设计方式。

* 选举三问：如果在一个term中，一个 Follower未收到一个Leader的心跳检测，那么之前为Follower角色的Server就会转变为Candidate角色。此时Server只是一个角色的转换吗？会开启一个新的term吗，因为Candidate此时会向集群中其他的Server发出选举请求。在文中提到一句一个Candidate或者Leader如果发现自己的term已经过时，此时Candidate、Leader就需要转变成Follower角色？
  
  * 一个如果在一个周期内（election timeout）没有收到Leader的心跳检测，此时Server就会从Follower角色转变为Candidate角色，同时升级自己的term。

  * 成为Candidate角色之后，该Server会给自己投一票，同时会向Cluster中其他的Server发送投票，此时应该也会带上自己的term。在选举二答中提到term的同步时通过服务器之间的通信会带上自己的term，同时其他Server在发现自己的term小于发起请求的Server的term就会升级自己的term，同时一个Leader或者一个Candidate发现自己的term小于其他Server的term时会从之前的角色转换成Follower角色。因此Leader在这个时候就会转换成Follower，转换之后的Follower就会响应Candidate的请求。


* 选举四问：因为是一个分布式系统，没有一个集中的决策机制，正是通过这种机制，那么一个Follower又是如何决定要给一个Candidate投赞成票还是投反对票。
  
  * 如果一个Candidate能够在集群中的同一个term的选举中获得大部分的Server的投票，那么这个Candidate就能够称为一个Leader。这里Follower如何给一个Candidate投票，可以在阅读了后main的内容之后再来回答这个问题。读了3.6Safety这一章节之后可以来回答了，比方说：

* 选举五问：如果一个Leader在规定的时间没有给用户发送心跳检测，那么这个Server会立即提升自己的term，也就是 Follower认为Leader已经挂掉了，Follower的状态会转换成Candidate，此时 $Candidate_{term} > Leader_{term}$。如果Candidate在选举过程中， Leader的网络状况没问题了，或者说Leader从宕机中重启了。Leader此时的状态应该发生什么变化，依旧可以认为自己是Leader还是转换成为Follower角色。
  * 这个问题和前面的选举三问中的问题一样，在选举过程中有一个很重要的点来判断一个Server的状态发生转换的就是一旦自己的term比其他Server的term要小，那么一定会从当前状态转换成Follower状态。
  
  * 因为Candidate的term应比上一轮的Leader的term要大，因此Candidate接受到Leader发送出去的AppendEntry RPC不做任何处理或者是返回失败码，同时告诉Leader已经在选举了。
  
  
* 选举六问：这个问题紧接着选举五问。集群中可能会出现同时存在Candidate选举和Leader执行log复制的现象。针对这种现象，Follower应该如何处理呢？我们假设两种情况。
  * case1：Follower的在没有收到Candidate之前，已经收到了Leader的AppendEtnry RPC 的情况下会出现集群中一部分的Server在复制旧的Leader的log，同时一部分Server在进行一部分选举，然后这部分Server在随后又收到了。
  * case2：Follower在接受到原来的Leader之前已经收到了Candidate的选举请求了，那么Follower会怎么做呢？**在Raft中有一个原则就是，服务器之间通信会互相交换term，如果其他Server的term比自己的大，那么会把自己的term升级到新的term，如果Candidate和Leader发现自己的term已经过时了，那么Candidate和Leader的状态转变成为 Follower**。有了这个原则，看一下Follower先收到Candidate的选举请求，此时 $Follower_{term} < Candidate_{term}$，因此Follower会升级自己的term，然后给这个Candidate投上一票。紧接着 Leader发起了Append Entries RPC，此时Follower发现$Follower_{term} > Leader_{term}$。那么此时Follower会忽略这个请求呢？还是会告诉Leader自己的term已经比他大了，Leader需要更改自己的状态为Follower还是说Leader要等到Candidate给Leader发送投票请求的时候Leader才将自己的状态转换成 Follower同时升级自己的term。这个问题我们留在下一个问题来解答。

* 选举七问：Leader在集群选举的同时给其他Follower和Candidate同时发去了AppendEntries RPC请求，请问此时其他Server应该如何响应。看一下论文里面有没有？
  * 在Raft 博士论文当中的Figure 3.1中明确指出了如果一个Follower在收到Leader的AppendEntries RPC请求，此时如果$Server_{term} > Leader_{term}$，此处用Server代替Follower和Candidate，此时Server 会返回自己的term，Leader发现自己的term已经过时了，因此会更新自己的term，同时因为此时这个Leader已经不再是Leader了，因此状态转换成Follower状态。
  

* 选举八问：当选举中是否会出现多个Leader（即分布式中著名的拜占庭将军问题），同时多个Leader是否最终能够由一个Leader来统领，Raft算法是如何解决此问题的。
  * 每个Follower都有自己的election timeout，而且这个每一个follower的election timeout的时间范围为[150,300]ms。那么这里就会出现150个不同的election timeout，如果时间更精确一点的话，可以有有无数的election timeout。

* 选举九问：选举中Candidate发出的请求和选举完成之后发出的心跳保活请求是否是同一个类型，那么Candidate和Follower角色的Server是如何做出区分的，同时又是如何回应的。

  * 在Raft的原文中提到了两种Rpc，一种是RequestVote主要是在当一个集群中的Follower无法感知到Leader时，此时Follower的角色就会转换成Candidate，此时就会使用RequestVote类型的Rpc与集群中其他Server进行通信。这个通信的目的是让其他 Follower知道自己已经是Candidate，投自己一票。还有一种是Entries Rpc，这种Rpc请求的方式就是Server将自己的log同步给Follower。

* 选举十问：那么这个时候衍生出一个问题，当一个集群正常运行时，集群中的Follower是如何知道其他 Follower的地址的。
  
  * 我想应该是当一个Candidate在成为Follower的同时，应该会发送一个Rpc请求，将集群中存活的所有Server的IP和Host同步给集群中所有的Follower。那么这中Rpc使用的是RequestVote Rpc还是使用Entries Rpc呢？如果是让我来设计，我应该如何设计呢？我想我会创建一个新的Rpc来解决这个问题。

* 选举十一问：加入一个集群中有5台Server，如果其中的Leader已经挂掉了，同时另一台 Follower也挂掉了，那么此时进入选举环节，其中剩余的每一台Server都会从Follower转换成Candidate角色。此时每一个Candidate都会给集群中的其他Server发送Vote Rpc（因为Candidate只知道Leader宕机了，并不知道另一台Server也宕机了，因此会发送出4个请求），此时Candidate首先会投自己一票，然后剩余的两票来自其他的Server，那么这个时候这个Candidate只得到的三票。这个时候这个Candidate还是可以获胜的，因为可以理解为5票中获取了3票，依旧是获取了集群中大部分集群的投票，此时Candidate会成为Leader。假设 Leader宕机的同时，另外还有两台Server也宕机了，此时两个Candidate会出现的情况是各自获得一票，或者是获得两票，那么此时Candidate会成为Leader吗？如果能够成为Leader是不是就默认集群中所有的Server都能感知到其他Server已经宕机的了，如果不是这种方式，那么又是如何解决的。
  
  * 这里从[Raft](https://raft.github.io/)官网可以看到这个如果集群中有五台Server，那么一旦有其中的三台Server宕机了，那整个集群就会陷入到无尽的选举过程，因为每一个Candidate都无法获得整个集群中半数以上（≥3）的投票，因此剩下的Server都会成为Candidate，成为Candidate的Server会不断的升级自己的 term。

* 选举十二问：如何防止一个Follower在选举期间只给其中一个Candidate投票，而不是给多个Candidate投票。
  * 在Raft论文提到了，每个term，Follower只给一个Candidate投票，因此不会出现一个Follower给多个Candidate投票的问题。
  * 

## 二、复制 - Replicate
* 复制一问：当一个集群在一个term开始阶段经历了选举完成，并选举出了一个Leader，那么此时Leader是如何接受Client的请求的，同时又是如何保证Client的请求在Leader和Follower机器上保证一致性。
  
  * Raft算法中，每台机器中包含一个 log文件，另外一个是state machine。log文件主要用于存储client的请求（这里暂且这样表述，因为在后面的集群中的配置也是存在log文件中），state machine主要用于执行log文件的命令。其中Leader在收到Client的请求时，首先是将用户请求中的Command写入到log文件中，Leader在写自己log文件的同时，也会通过AppendEntries RPC请求将Command复制给集群中的Follower。
  * Leader在收到Follower的响应之后，就开始让state machine执行log文件中的command，并将执行到的结果返回给Client。
  * 当Follower知道Leader已经提交了同一个位置的entry之后，Follower同样也会将同一个位置的index交给state machine执行。

  
* 复制二问：集群中只有一个Leader，有多个Follower，那么多个Follower是全部都写入log之后Leader才执行command还是只要集群中的半数以上的Follower已经将command写入log就可以执行Command然后返回结果。
  
  * Raft Thesis中提到log中的一个entry被state machine执行后称之为commit（提交）。 log中每一个entry包含的信息有command和term。Leader只要集群中大多数的Follower已经复制了entry之后，就会提交这个entry（也就是将entry交给 state machine执行）。

* 复制三问：响应一个Client的请求分为三个步骤，Leader首先写自己的log，然后Leader将entry同步给Follower，Follower将entry写入到自己的 log；第三步Leader将entry提交到state machine执行，第四步Follower了解到Leader已经提交了entry，那么此时Follower也会将这个entry在自己的state machine中执行。
  
  * 这里每一步都有可能出错，如果第一步Leader在写自己的log成功之后，此时Leader宕机；在第二步一部分Follower将entry写入到log之后，此时Leader宕机；在第三步，大部分Follower将entry写入到log之后，此时Leader使用state machine执行command之后宕机；第四步，一部分机器在提交entry的时候成功，一部分机器执行entry失败，在后续的选举或者是同步过程中，应该如何处理这部分数据。这里也可以看到在分布式系统中，一个步骤被拆的越详细，出问题的概率也会越大。
  
  * 针对这几个问题，会涉及到后续Leader选举时应该选谁当Leader问题（因为一部分 Follower已经将entry写入log当中），选举完成之后Leader与Follower之间的 log同步问题（因为一部分Follower写人了这个entry，而另外一些Follower并没有写入）；第三个问题就是Client请求的幂等性问题（如果Leader在提交之后宕机，那么下一次client在请求的时候就会很迷惑，比方说现在客户看到了x = 1,在刚才宕机的时候要求执行x++，此时Leader提交了但是Leader宕机了，那么客户看到的就是异常响应）。

* 复制四问：如果当一个选举完成之后，返现Leader与Follower之间的log不一致怎么办？
  
  * 如果Leader和Follower之间的log不同，因为遵循Log Matching Property，根据Log Matching Property，Leader和Follower之间的log在同一个位置的term和command一致，如果不一致，那么就需要在Leader和Follower之间找到entry相同的位置，然后Follower删除全部不相同的entry。Follower通过这种方式和Leader之间保持一致。
  * 具体操作流程，如果一个Candidate成为一个Leader，那么Candidate会把自己最后一个entry的数据和Follower之间进行匹配（注意，这里第一步时匹配，Leader一旦选举出来并不是马上开始同步自己的entry给Follower）。如果 Follower和Leader在同一个位置的log不匹配，那么Leader可以和Follower之间通过多次通信的方式进行
  


## 三、安全 - Safety
1、选举补充
* Raft文章表述只允许集群中那些包含了上一轮已经提交的 entry的Server才能获取其他Server的投票。这是为什么呢？
  * 如果一个request在当前term，我们假设好似$term_T$已经被提交了，根据之前一个entry被提交的原则，一定是entry已经被集群中半数以上的Follower复制了这个entry。那么为什么要有这样一个原则呢？
  * 我想是因为如果一个request已经被请求，同时已经被提交了，那么当Leader宕机，集群在下一个$term_{T+1}$时刻选举出一个新的Leader，如果这个Leader没有包含这个entry，那么这个request肯定是需要被重新执行一次。但是如果这个Leader在上一轮已经复制了，那么可以直接返回结果给Client。
  
* 如果一个集群中有一半的Server已经复制了entry，那么在当前term下，如果Leader宕机，那么其他机器没有获得心跳检测，如果是那些没有完成最新entry提交的Server因为最先感知到Leader发送的心跳检测，那么这些机器就会成为Candidate，根据前面提到的原则，这一轮这个Candidate肯定是不能获得集群中的半数以上的投票的，那么在下一轮投票开始前，是上一轮Candidate继续保持Candidate状态还是有新的 Server从Follower状态成为Candidate状态？
  * 这个问题应该怎么回答呢？我理解如果在当前轮Candidate无法获得半数以上的投票，那么次轮投票结束，因此下一轮大家都会升级自己的term，然后每一个机器都会成为Candidate角色。如果是按照这种假设。
  * 那么其他原先为Follower的Server是如何知道这轮投票结束的，因为之前我们只是说如果Server在Follower状态在election timeout内没有收到Leader的心跳检测请求，就会从Follower状态转变成为Candidate状态，现在是进入投票环节，那么Follower是如何从Candidate状态转变成为Candidate状态呢？如果没有这个环节，那么整个集群永远也选不出一个合格的Leader。 
  * 在试用了[Raft](https://raft.github.io/)，对于上面的问题又了一个初步的答案，如果一个慢的Server（就是复制Leader的log比较慢）成为了Candidate，其他Follower在收到慢Server的RequestVote RPC的时候，其他机器会进入到timeout election，然后这些机器一旦election timeout后就会进入Candidate状态。
  * 如果Server在收到AppendEntries RPC或者RequestVote RPC时，这时Follower会重新计时，如果在Follower重新计时结束后还没有收到Leader的AppendEntries RPC或者还没有收到Candidate发出的RequestVote RPC，就会自动转换成Candidate状态。
  * 如果一个Follower收到的Candidate的RequestVote RPC发现Candidate的log还没有自己长（已提交的entry长度），如果这个Server已经提交了这个entry，那么肯定是集群中有大半Server已经提交了，那么这个Candidate是无法获取到集群中半数的投票，所以集群中大半Server会给这个Candidate投反对票，然后大家都进入 election timeout倒计时，计时结束后，最先计时完毕的Server状态会从之前的Follower转变成为Candidate状态，然后给其他Server投票。其实从这个机制就很好的保证了不会有slow Server一直是Candidate参与选举，其他Server也能保证自己能够成为Candidate被选举成为Leader。
  * 这里还有一个疑问就是如果Slow Server成为Candidate，那么在选举过程中被大多数Server投了反对票是等投票结束以后，然后重新进入election timeout计时还是说投票过程也算是election timeout。就是说此时Candidate的投票计时和election timeout是否共用一个计时。
  ![avatar](/分布式/img/slow-server-candidate.png)
  其实在这种情况下，当Follower开始进入到election timeout环节时，对于已经拒绝的RequestVote RPC是不会重新计时的，也就是继续当前计时，计时结束还没有收到其他AppendEntries RPC时，就会进入Candidate状态。那么我们还需要看一下如果时给其他符合条件的Candidate投票，此时应该如何处理。从官网的展示来看，如果是正常投票，那么其他Follower会进入重新倒计时。
  
* 如果一个集群中只是单纯的以某一个Follower因为election timeout，就进入Candidate状态，然后开始选举，那么集群中只要有慢的


2、复制补充

## 四、集群成员变化
## 五、Log压缩
完成Raft第五章的学习并把第四章类容回顾一下

* 问题一：什么是Log compaction？

  * 回顾一下今天的主题，为什么要做Log Compaction，是因为当Leader接受到更多了Request之后，每一台Server要存储的Log会变得越来越多，如果是存储在内存当中，随着Client请求越来越多，需要存储的entry会变多，内存有限。一旦entry被提交之后，其实entry就已经不再被需要需要将一部分历史数据通过Compaction存储到disk。
  * 什么是 Log Compaction呢？简而言之，就是Raft中的一些entry已经过时了，需要丢弃掉，只需要保存系统的最终状态就可以。
  * 什么是snapshotting？将Log中一段时间的entry通过序列化的方式写入到 disk后，将中间状态丢弃，然后只存储本次压缩后的系统的状态以及log的位置、集群最新的配置信息。其中state machine负责将Raft中的log从内存中序列化后写入到disk当中。同时Raft需要保存本次写入到disk的最后一个entry的index、term、系统最终状态、集群成员配置等信息。
  
  * 我们详细讲一下 Log compaction，我们假设log的entry为(index = 1, term = 1, x = 1),（index = 2, term = 1, x = 2），(index = 3, term = 2,x = 3)，(index = 4, term = 3, x = 5)，(index = 5, term = 4, x = 7)，如果Raft决定在index = 4时执行log compaction，此时Raft就会将log复制到state machine当中，然后state machine负责将index = 1位置至index = 4位置的数据序列化后写入到disk，然后Raft存储一个数据，这个数据就是index = 4这个位置时的系统最终态（我们这里只列举了index、term、x，这里的变量只有x，其实在真实的分布式系统中会有很多，我们这里也忽略了Cluster Member Info）。然后Raft就会将index = 4 这个位置和之前的数据全部删除，log中剩下的就只有(index = 5, term = 4, x = 7)。下一次要执行log compaction的时候，只需要从这个位置开始就可以的(其实这里也有点想不明白，如果采用array的话，那么就算前面的数据已经被删除了，那么删除了的entry的数组空间能够被释放吗，这里还是存有疑问的，如果只是单纯的删除了元素，并没有释放空间，那么这个操作的意义是啥呢？这里在具体实现中希望能够找到一个解答)。
  
  * 这里具体的流程和细节可以参考Raft thesis论文中关于Log Compaction的细节，我们这里主要关注序列化的几个问题，一个是Log Compaction一方面需要占据State Machine的执行，那么如何保证在能够完成client请求的情况下，同时能够完成log compaction。第二个问题就是Raft应该在什么时候开始执行Log compaction？第三个问题是序列化的数据如何写入到Disk中，因为后续存在slow follower（集群中复制比较慢的Server）和new Server（新加入到集群中的Server），这些Server在复制Leader的log时，Leader可能已经把这些Server要复制的index位置的entry已经从log中删除了，那么如何从之前log compaction操作写入到disk的snapshot（就是序列化好的一段时间的entry集合）恢复出来，并将这些entry能够同步给slow Server和new Server。
  * 这里还有一点值得提出的就是，想比较log replication是从Leader同步entry到Follower，log compaction操作是每个Follower自己决定的，只有一个原则，机器能够在宕机重启后恢复出宕机之前已经复制好的entry和state就可以。

* 问题二：对于Leader中已经删除的log，那么针对 slow Server和new Server应该通过同步呢？
  * 在前面关于Cluster MemberShip Change的时候，Leader一方面需要将集群信息写入log然后通过AppendEntry RPC同步给集群中其他Server（Follower）。同时新加入的Server如果想要加入Cluster，那么需要确保这个Server能够足够快的复制entry。现在出现log compaction，Leader只保留最新的system state（系统各个变量的最终值），此时如果new Server想要从Leader中复制这个index之前的entry，那么只能从disk中找到包含这个index的snapshot，然后将反序列化后对应index位置的entry同步给new Server和slow Server。

* 问题三：如果Leader已经删除了自己的log，同时将历史序列的entry已经序列化写入到disk了，此时如果机器宕机了，服务重启之后应该如何恢复之前的状态呢？比方说log信息如何恢复？
  
  * 这里可以将log分为两种情况，如果一个 log的数据已经全部写入到disk了，那么是否需要加载所有的 snapshot之后才能复现宕机之前的state呢？我想是不用的，因为如何这么做的话，这个系统设计就太傻了。
  * 如果宕机的时候，log有一部分在h还没有删除，一部分已经写入到disk了，那么这一部分就需要怎么操作呢，因为log中已经存在的数据能够保证是服务器当前的状体。


* 问题三：什么时候将log中的entry序列化并将序列化的数据写入到disk当中？
  
* 问题四：应该以何种数据结构存储序列化的数据并且能够实现快速写入和读出数据？
  
  * state machine将log存储在内存中，将log在某一个时间点之前的entry全部写入到stable storage上，将已经写入的entry从现在log上删除。
  
  * 有的state machine是将状态存储在disk上，一旦这个raft的log已经写到磁盘上，那么这个log就可以被删除。
  

  * Raft负责将某一个时间节点的log的prefix（之前的所有entry）转移到 state machine（这里的转移可以理解为复制）。state machine负责将这些entry写入到disk（其实是需要将这些entry完成序列化，然后写入disk），有了这个操作，一旦机器宕机了，也可以从disk中读取当前的系统的state，从而恢复机器的状态。Raft在放弃之前的entry之前，需要将之前的entry进行一个压缩，比方说中间的entry有x=3,但是在log Compaction节点时刻最后一个entry的状态是x = 5, 此时log就会将存储x = 5，之前的状态全都丢弃。我很好奇，如果Raft保存这个信息，那么Raft保存在哪里呢，是不是说这个其实就是一个Flag，告诉下一次执行log compaction的时候需要从何处开始。我之前以为的log compaction是将最后的状态写入的磁盘，其实这里是有一个误区的，写入到磁盘的是log entry未被压缩的信息，Raft是在entry写入磁盘后，将某一个节点的数据全部清除，然后保存最后状态便于下一次执行log compaction能够找到位置。
  * 之前在提到Cluster Configuration Change的时候会把 Cluster Information存储在log上面，Log Compaction之后Raft也会保留最新的Cluster Membership Information；
  
  * Raft一旦log compact，就会将之前的entry丢弃，state machine就需要承担两个事情，一个是服务重启时，state machine需要从磁盘中加载之前log compaction 操作执行的时候存储的system state信息恢复log。
  * 一个集群中可能会出现slow follower，这些follower复制entry的速度比较慢，如果Leader已经执行了log compaction，那么这些较慢的follower在执行log replication的时候会出现这个entry在Leader侧已经丢弃（因为Leader已经执行了log compaction）。那么state machine此时就需要将存储在disk的state重新恢复出来，然后让slow follower能够完成 log replication（其实这个slow follower大概率是新加入集群的Server）
  * 其实一个log compaction，将数据存储到disk，也是需要考虑到存储数据的数据结构。如果一个slow follower需要执行一次同步但是Leader的log已经删除，那么state machine就需要快速的从已经存储在disk找到对应的信息。所以这个数据结构要有很好的查找和写入的能力，而不能只是单纯的将其写入，而查找却耗费大量的时间。
  * 一旦state machine完成了写入一个snapshot完成，那么log就可以丢弃，Raft就需要存储snapshot时刻的index和term，以及这个位置是每个变量的最新的状态，以及最新的集群配置信息。
  * snapshoting这里有个问题就是一方面需要解决的问题就是应答client和执行序列化和写入disk需要并行执行。而且在执行序列化的时候有可能将内存全部用完的情况。
  * 服务器什么时候执行snapshotting呢？一旦当前的log的大小已经超出了前一个snapshot文件一定的量级之后就需要执行快照存储了。
## 六、Client请求
论文作者把用户请求集群的细节放到了论文的最后部分解答，因为这个不是这个分布式系统最需要关注的问题。Client请求可以简单的理解为两台机器之间的通信，那么只需要知道两台机器的IP和host，那么两台Server就可以建立连接。那么为什么这样一个简单的事物化为什么会单独拿出一章来讲这个看似简单的问题呢？一开始以为这一章只是一个简单的问题，其实这一章和我们工作中接触到的Mysql、Redis等分布式数据存储都是息息相关的。如果能够很好的回答这些问题，其实对于分布式系统大多数问题都能够得到解答。

* 如果一个集群是不变的，同时Leader的信息不发生变化的情况下，那么这个问题就简化了，因为只要Client和Server之间建立连接就可以通信了。现实情况是集群的Leader会发生变化，因此Client需要保证自己能够和Cluster进行通讯。
  
* 一个集群中Cluster 成员会发生变化，因此我们实际上在连接的时候从来使用的不是ip:host方式进行连接的，而是使用域名的方式进行连接。那么业务中使用域名方式又是如何解决集群中Leader信息发生变化的的？这个问题其实也要好好的看一下。

* client与Cluster集群之间建立连接后，需要解决的另外一个问题如何保证Client的一个请求只会被commit一次，而不会被执行两次，执行两次是什么情况呢？如果Client第一次和Leader进行交互，如果此时Leader已经让集群中大多数Server复制了entry（针对这次请求的entry），同时Leader也将entry提交给了state machine，但是在state machine执行完成之后Leader宕机，此时无法回应用户的请求。当集群下一次选出一个新的Leader，Client与new Leader重新建立连接，此时Client再一次请求执行之前的操作， new Leader会把之前已经执行过的command重复执行一次，相当于client只想针对变量x = 1自增1，但是因为服务器的崩溃导致服务器执行了两次，此时x = 3了。Raft还需要解决写一致性。
* 如果一个分布式系统存在写一致性，那么也会存在读一致性，这个问题应该如何解决呢？

## TODO List
* 回答一些今天在写PPT的时候遇到的问题
* 完成Raft论文笔记的重构
  
## 学习资料
[How I am learning distributed systems](https://medium.com/@polyglot_factotum/how-i-am-learning-distributed-systems-7eb69b4b51bd)
 
[The Raft Consensus Algorithm](https://raft.github.io/)



