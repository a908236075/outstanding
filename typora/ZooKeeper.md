# 分布式过程协同技术详解

## 第一部分 概念和基础

1. 概念:开源的分布式协同服务.
2. 设计目的
   - 高性能:类似于文件的系统的命名方式,将所有路径放在内存中,可以进行快速访问.
   - 高可用:可以搭载集群以及发生故障后自动进行的主节点选举.
   - 严格的顺序性:客户机上可以实现复杂的同步.
3. 使用范围:整个ZooKeeper的服务器集群管理着**应用协作的关键数据**。ZooKeeper不适合⽤作海量数据存储。是将分布式常遇见的消息延迟导致的数据不一致,硬件服务器性能引起的假死等问题在**应用层面上完全透明化**.
4. 一般的主-从应用
   1. 主节点崩溃:不能进行任务的分配.重新选举主节点,同时避免脑裂的发生.
   2. 从节点崩溃:不能讲分配的任务正常的执行.从新分配任务,并恢复清除受到影响的过程.即源数据管理和成员关系的管理.
   3. 通信故障:崩溃监测,主节点必须具有检测从节点崩溃或失去连接的能力.
5. ZNode节点的不同类型:
   1. 持久节点:调用删除方法的时候才会删除.
   2. 临时节点:创建改节点的**客户端**崩溃或者关闭了与ZooKeeper的连接,这个节点就会删除.
   3. 有序节点:每次创建都会有一个序号和持久与临时节点组成四个节点类型.
6. 监视与通知
   1. 每次都要客户端请求获取这种轮询的方式性能不好,所以采用了服务端通知的方式.
   2. 每次通知都是触发了一个监视点.
1. ![通过通知的方式进行znode的变化](D:\develop\GitHub\project\outstanding\typora\picture\zookeeper\通过通知的方式进行znode的变化.png)
      2. 当第三部的时候,zookeeper又有新的变化,这时客户端会获取最新的值,以保证不会错过状态的变化.
7. 版本号
   1. 每⼀个znode都有⼀个版本号，它随着每次数据变化⽽⾃增.
   2. 当调用setData和delete的时候,版本号作为参数进行传递,zookeeper会校验版本号, 如果不一致执行就不会成功.
8. ZooKeeper的架构
   1. 法定人数:是指为了使ZooKeeper⼯作必须**有效运⾏的服务器的最⼩数量**。避免因等待所有数据同步的延时问题.
   2. 会话采用的是TCP协议的连接通信.
   3. zookeeper集群对声明会话超时负责,而不是客户端.即客户端不能对会话进行超时管理,但是可以选择关闭会话.
   4. 客户端请求将全部以FIFO顺序执⾏。如果客户端拥有多个并发的会话，FIFO顺序在多个会话之间未必能够保持
   5. 当客户端进行连接的时候,会获取可以连接的zookeeper的服务器的列表,连接的服务器ZooKeeper状态要与最后连接的服务器的ZooKeeper**状态保持最新**.
   6. 客户端连接成功后,会得到服务器给的zkid,通过这个标识进行下一次的会话通信,如果断开了连接,也通过这个标识进行重新连接,如果服务器给的新的zkid小于之前的,则不成成功建立会话.
   7. **锁**:为了获得⼀个锁，每个进程p尝试创建znode，名为/lock。为了避免出现死锁,我们不得不在创建这个节
   

点时**指定/lock为临时节点**。
## 第二部分 使用zookeeper进行开发

1. ### Zookeeper的API

2. ### 监控

   1. ZooKeeper的API中的所有读操作：getData、getChildren和exists，均可以选择在读取的znode节点上设置监视点。
   2. 监视数据的变化和监视子节点的变化.
   3. 某一时刻设置了大量的监视点,会导致尖峰的通知,解决的办法是客户端通过/getChildren方法来获取所有/lock下的⼦节点，并判断⾃⼰创建的节点是否是最⼩的序列号。如果客户端创建的节点不是最⼩序列号，就根据序列号确定序列，并在前⼀个节点上设置监视点。
      1. 创建/lock/lock-001的客户端获得锁。
      2. 创建/lock/lock-002的客户端监视/lock/lock-001节点.
      3. 创建/lock/lock-003的客户端监视/lock/lock-002节点。
   4. 当一个客户端连接到一个新的服务器上时，watch 将会被以任意会话事件触发。当与一个服务器失去连接的时候，是无法接收到 watch 的。而当 client 重新连接时，如果需要的话，所有先前注册过的 watch，都会被重新注册。通常这是完全透明的。只有在一个特殊情况下，watch 可能会丢失：对于一个未创建的 znode的 exist watch，如果在客户端断开连接期间被创建了，并且随后在客户端连接上之前又删除了，这种情况下，这个 watch 事件可能会被丢失。
   5. 注册watch
      1. getData、exists、getChildren
   6. 触发watch
      1. create、delete、setData
   7. 顺序的保障
      1. **隐藏通道**的问题:在Zookeeper之外,服务端和客户端还保有通信,导致直接从c1中获取的数据是过期的.通过设置C1的监控点,而非直接从C1中获取数据来避免.

3. ### 故障处理

   1. 当服务端发生故障的时候,会给客户端返回连接异常的状态码,由客户端去自己去处理异常,客户端首先要将交互的所有动作挂起,以避免数据不一致的情况.需要注意,一旦发生异常,客户端就没有办法显示的关闭会话.如果网络或者其他问题导致的延时足够长,服务端就会自己关闭会话.
   2. 不可恢复的故障:处理不可恢复故障的最简单⽅法就是**中⽌进程并重启**，这样可以使进程恢复原状，通过⼀个新的会话重新初始化⾃⼰的状态。如果该进程继续⼯作，⾸先必须要清除与旧会话关联的应⽤内部程状态信息，然后重新初始化新的状态。
   3. zookeeper 无法限制资源外的交互,所以创建了对客户端资源外的**群首处理**.始终由一个客户端保证**资源的一致性**,为了实现,需要所有的客户端**维护一个zxid**,如果因**时钟偏移(超载使时间暂停)或者过载**使得客户端操作资源超时,在从新确定了群首之后,第一群首的请求才被接受到,就会比对zxid,如果小,则此操作舍弃不执行.

4. ### 注意事项

   1. zookeeper提供了4种内置的模式进行ACL的处理,通过**OPEN_ACL_UNSAFE**常量隐式传递了ACL策略，这种ACL使⽤world作为鉴权模式，使⽤anyone作为auth-info，对于world这种鉴权模式，只能使⽤anyone这种auth-info。

## 第三部分

### zookeeper的内部原理

1. zxid为⼀个long型（64位）整数，分为两部分：时间戳（epoch）部分和计数器（counter）部分。
2. 群首选举
   1. ![](D:.\picture\zookeeper\群首选举.png)
3. Zab状态更新广播协议——ZooKeeper原⼦⼴播协议（ZooKeeper Atomic Broadcast protocol）
   1. 群⾸向所有追随者发送⼀个PROPOSAL消息p
   2. 当⼀个追随者接收到消息p后，会响应群⾸⼀个ACK消息，通知群⾸其已接受该提案（proposal）。
   3. 当收到仲裁数量的服务器发送的确认消息后（该仲裁数包括群⾸⾃⼰），群⾸就会发送消息通知追随者进⾏提交（COMMIT）操作。
4. 为了阻⽌系统中同时出现两个服务器⾃认为⾃⼰是群⾸的情况是⾮常困难的，时间问题或消息丢失都可能导致这种情况，因此⼴播协议并不能基于以上假设。为了解决这个问题，Zab协议提供了以下保障：
   1. ⼀个被选举的群首确保在提交完所有之前的时间戳内需要提交的事
      务，之后才开始⼴播新的事务
   2. 在任何时间点，都不会出现两个被仲裁支持的群首。
5. 如果群首在提交事务后崩溃,并选举产生了新的群首,那已经通知了从节点的服务器,仍然会根据zxid的前32时间戳来判断执行老群首所**提交**的事物,保证事物不会被遗漏.
6. 观察者和追随者一样,不同的是不参与投票.

配置相关以后再去学习   

未完待续.......