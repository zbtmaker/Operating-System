# 简易RPC框架
一个人如果完全成熟了，那么这个人的问题很少，并不是说一个问问题少的人是一个成熟人，而是他的经验验足够丰富。世上这一类人可能很少，每个人都会有自己的知识盲点，所以大部分人应该都是一生都在成长的，不是某一个节点就豁然开朗，而是在接触这个世界新事物的时候都会感受到新奇。一个人如果在成长，那么这个人是会问问题的，问题的背后代表这人对问题思考，而问这个动作就是需要一个外界的声音回应这种思考。

一个框架的背后是一种通用规则，通过一种规则来实现自己的想法，同时并不只支持一种规则，而是让众多规则都能够适用。从特殊到一般的思想，一切复杂的问题必然是简单的，而一切简单的问题也必然是复杂的抽象。

## 问题
* 在client端或者在server端为什要给调用和被调用需要创建代理对象才可以。
* 之前一直以为client端和server端之间请求和响应是需要经过注册中心的，原来client和server之间的请求和响应是直接连接方式。
* 这里有一个疑问就是如果在client请求的瞬间，与这个client连接的server在这一瞬间宕机了，那么client怎样来处理呢？是通过注册中心获取其它的server，同时保证宕机的服务器不会出现在这个列表里面，或者这个server出现在这个列表里面，但是有一个失败的列表，因此不会重复请求，那么还有一个问题就是假设这个集群中的一半机器宕机，怎么保证在一次请求失败之后能够快速找打一个仍然还在服务的server。
* 如果client和server是直接连接，在client和server在启动后都取到了对方的信息（其实流程是server向注册中心注册自己的信息，client向注册中心拉取订阅的服务信息），假设client和server都完成了信息获取，这个时候注册中心zookeeper集群宕机，那么client和server之间还可以继续通信吗？
* 负载均衡是谁来决定的呢？是注册中心还是client在请求的时候需要保存这些数据。
* client和server与注册中心之间是如何交互的呢？比方说注册中心通过哪种方式。
* 这里有个问题就是我们如何保障我们的client和server之间的协议是一致的，那么如果协议不一致，框架能够兼容吗？如果client和server之间处在不同的协议如果已有的框架没有提供这样的功能，约束这个问题的原因是什么呢？比方说client升级了框架的版本，在框架的版本上新增了一种protocol，但是server并没有增加这个protocol，这种case是否会报错，是理应报错还是说可以兼容。

## 阅读





