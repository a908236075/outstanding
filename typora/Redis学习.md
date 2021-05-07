**Redis定义**:

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件

~~~shell

redis-cli --cluster fix 127.0.0.1:7000
~~~

----

# Redis 深度历险 核心原理和应用实战

## 基础知识

1. 准备环境

   1. ~~~shell
      # 拉取 redis 镜像
      > docker pull redis
      # 运行 redis 容器
      > docker run --name myredis -d -p6379:6379 redis
      # 执行容器中的 redis-cli，可以直接使用命令行操作 redis
      > docker exec -it myredis redis-cli
      ~~~

2. Redis的所以的key都是String类型的.不同的是value值的类型不一样.

3. 基础数据结构:String,list,hash,set,zset.
   1. String:Redis 的字符串是动态字符串，是可以修改的字符串，内部结构实现上类似于 Java 的
      ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.
      
      1. ~~~shell
         # 常用命令
          set name  zhangsan
          get name
          exists name
          del name
          ## 多值
          mset name1 lisi name2 wanger
          mget name1 name2
          ##过期
          expire name 5 ## 对已有字段设置 5s 后过期
          setex name 5 codehole # 5s 后过期，等价于 set+expire
          setnx name codehole # 如果 name 不存在就执行 set 创建
         ## 计数
         set age 30
         incr age # 31
         incrby age 5 #36
         incrby age -5 # 31
         ~~~
      
   2. List:相当于 Java 语言里面的 LinkedList，注意它是链表而不是数组,增删快,查找慢.

      1. ~~~shell
         ## 右边进 左边出 队列
         rpush books python java golang
         llen books #3
         lpop books # python 消耗了一个books里面的元素
         ## 右边进 右边出 栈
         rpush books python java golang
         rpop books # golang  可以一直执行这个语句 直到取完
         ~~~

      2. ~~~shell
         ## 范围操作
         > rpush books python java golang
         (integer) 3
         > lindex books 1 # O(n) 慎用
         "java"
         > lrange books 0 -1 # 获取所有元素，O(n) 慎用
         1) "python"
         2) "java"
         3) "golang"
         > ltrim books 1 -1 # O(n) 慎用
         OK
         > lrange books 0 -1
         1) "java"
         2) "golang"
         > ltrim books 1 0 # 这其实是清空了整个列表，因为区间范围长度为负
         OK
         > llen books
         (integer) 0
         ~~~

   3. Hash(字典):Redis 的字典相当于 Java 语言里面的 HashMap，它是无序字典。内部实现结构上同Java 的 HashMap 也是一致的，同样的数组 + 链表二维结构。不同的是rehash是**渐进式的**.k可以用来保存用户信息,避免想String 那样一次性将所有的字段取出来.

      1. ~~~shell
          hset books java "think in java" # 命令行的字符串如果包含空格，要用引号括起来
         (integer) 1
         > hset books golang "concurrency in go"
         (integer) 1
         > hset books python "python cookbook"
         (integer) 1
         > hgetall books # entries()，key 和 value 间隔出现
         1) "java"
         2) "think in java"
         3) "golang"
         4) "concurrency in go"
         5) "python"
         6) "python cookbook"
         > hlen books
         (integer) 3
         > hget books java
         "think in java"
         > hset books golang "learning go programming" # 因为是更新操作，所以返回 0
         (integer) 0
         > hget books golang "learning go programming"
         > hmset books java "effective java" python "learning python" golang "modern golang
         programming" # 批量 set
         ~~~

   4. Set:Redis 的集合相当于 Java 语言里面的 HashSet，它内部的键值对是无序的唯一的。它的
      内部实现相当于一个**特殊的字典**，字典中所有的 value 都是一个值 NULL。

      1. ~~~shell
         > sadd books python
         (integer) 1
         > sadd bookspython # 重复
         (integer) 0
         > sadd books java golang
         (integer) 2
         > smembers books # 注意顺序，和插入的并不一致，因为 set 是无序的
         1) "java"
         2) "python"
         3) "golang"
         > sismember books java # 查询某个 value 是否存在，相当于 contains(o)
         (integer) 1
         > sismember books rust
         (integer) 0
         > scard books # 获取长度相当于 count()
         (integer) 3
         > spop books # 弹出一个
         "java"
         ~~~

   5. Zset (有序列表):它类似于 Java 的 SortedSet 和 HashMap 的结合体，一方面它是一个 set，保证了内部value 的唯一性，另一方面它可以给每个 value 赋予一个 score，代表这个 value 的排序权重。它的内部实现用的是一种叫着「跳跃列表」的数据结构。应用与有排序需要的场景,例如按照关注事件对粉丝进行排序.

      1. ~~~shell
         > zadd books 9.0 "think in java"
         (integer) 1
         > zadd books 8.9 "java concurrency"
         (integer) 1
         > zadd books 8.6 "java cookbook"
         (integer) 1
         > zrange books 0 -1 # 按 score 排序列出，参数区间为排名范围
         1) "java cookbook"
         2) "java concurrency"
         3) "think in java"
         > zrevrange books 0 -1 # 按 score 逆序列出，参数区间为排名范围
         1) "think in java"
         2) "java concurrency"
         3) "java cookbook"
         > zcard books # 相当于 count()
         (integer) 3
         > zscore books "java concurrency" # 获取指定 value 的 score
         "8.9000000000000004" # 内部 score 使用 double 类型进行存储，所以存在小数点精度问题
         > zrank books "java concurrency" # 排名
         (integer) 1
         > zrangebyscore books 0 8.91 # 根据分值区间遍历 zset
         1) "java cookbook"
         2) "java concurrency"
         > zrangebyscore books -inf 8.91 withscores # 根据分值区间 (-∞, 8.91] 遍历 zset，同时返
         回分值。inf 代表 infinite，无穷大的意思。
         1) "java cookbook"
         2) "8.5999999999999996"
         3) "java concurrency"
         4) "8.9000000000000004"
         > zrem books "java concurrency" # 删除 value
         (integer) 1
         > zrange books 0 -1
         1) "java cookbook"
         2) "think in java"
         ~~~

      2. 数据结构:zset 内部的排序功能是通过「跳跃列表」数据结构来实现的,类似于公司的职能部门:
      
   6. 

   7. 

## 应用

1. 分布式锁

   1. 导致Redis分布式锁失效的原因是在加锁和执行的结果的命令分为了两条命令执行,不能保证原子性解决的办法是将set命令多加了参数,使他们变成一条指令

      1. ~~~shell 
         >set lock:codehole true ex 5 nx OK ... do something critical ... > del lock:codehole 
         ~~~

      2. 上面这个指令就是 setnx 和 expire 组合在一起的原子指令

   2. 超时问题:可以使用在set赋值添加随机数,如果随机数正确了才将锁释放,这个步一般在代码中完成.

   3. 不建议使用可重入锁.

2. 延时队列

   1. 好处:适用于只有一组消费者的消息队列,实现简单,不用向其他消息队列那样要创建主题,分组还要进行连接,这种性能的损失是没有必要的.
   2. 队列延迟:队列延迟,如果队列为空,客户端一直做pop无意义的动作,**用阻塞队列代替队列**,阻塞读在队列没有数据的时候，会立即进入休眠状态，一旦数据到来，则立刻醒过来。消息的延迟几乎为零。用 blpop/brpop 替代前面的 lpop/rpop.
   3. 实现:可以通过zset的来实现.我们将消息序列化成一个字符串作为 zset 的 value，这个消息的到期处理时间作为 score，然后用多个线程轮询 zset 获取到期的任务进行处理.

3. 位图

   1. 一种为了节约空间以位作为操作单元的结构.
   2. setbit word 4 1 给word变量的第4位设置为1.
   3. bitfile 可以一下执行多个命令.

4. HyperLogLog可以提供不精确的去重方案.

5. 布隆过滤器:布隆过滤器可以理解为一个不怎么精确的 set 结构.基本指令:bf.add 添加元素，bf.exists 查询元素是否存在.

6. 限流 Redis-Cell 使用唯一的命令 cl.throttle可以限制每秒的允许请求数.

7. scan 代替keys,不会阻塞线程,可以使用limit.

   1. ~~~shell
      scan 0 match key99* count 1000
      ~~~

   2. scan 参数提供了三个参数，第一个是 cursor 整数值(开始值的索引)，第二个是 key 的正则模式，第三个是遍历的 limit hint。limit 是 1000，但是返回的结果只有 10 个左右。因为这个 limit 不是限定返回结果的数量，而是限定服务器单次遍历的**字典槽位数量**(约等于)。

   3. redis-cli 有大key扫描的命令.

## 原理

1. redis是单线程的.用到了非阻塞队列和多路复用等技术,使得性能非常的好.
2. Redis快照:Redis 使用操作系统的多进程 COW(Copy On Write) 机制来实现快照持久化.
3. 读和写真正耗时的地方:如果发送缓冲满了,那么就需要等待缓冲空出空闲空间来，这个就是写操作 IO 操作的真正耗时.但是如果缓冲是空的，那么就需要等待数据到来，这个就是读操作 IO 操作的真正耗时。
4. 序列化协议
   1. 单行字符串 以 + 符号开头。
   2. 多行字符串 以 $ 符号开头，后跟字符串长度。
   3. 结尾为/r/n
5. 持久化 Redis 的持久化机制有两种，第一种是快照，第二种是 AOF 日志.
6. 结合了两种机制的第三种,混合持久化,将 rdb 文件的内容和**增量的 AOF 日志文件**存在一起。
7. fork 多进程,redis做持久化的时候,会fork子进程,子进程刚刚产生时，它和父进程共享内存里面的代
   码段和数据段,进行持久化.而父进程有时候需要修改数据.进程这个时候就会使用操作系统的 **COW(Copy On Write) 机制**来进行数据段页面的分离。数据段是由很多操作系统的页面组合而成，当父进程对其中一个页面的数据进行修改时，会将被共享的页面复制一份分离出来，然后对这个复制的页面进行修改。这时子进程相应的页面是没有变化的，还是进程产生时那一瞬间的数据.
8. 事务:redis的事物只是使命令按照顺序执行,即使中间的命令出错,也不会影响后面的命令,redis的事务仅仅满足了隔离性,而没有满足原子性.
9. 分布式锁是一种悲观锁,而使用watch可以实现乐观锁.
10. redis做消息队列的时候不足之处是**不支持消息的多播机制.**为此创建了PubSub,但是如果 Redis 停机重启，PubSub 的消息是不会持久化的，毕竟 Redis 宕机就相当于一个消费者都没有，所有的消息直接被丢弃。正是因为 PubSub 有这些缺点，它几乎找不到合适的应用场景。为此创建了Stream,弥补不能持久化消息的问题.
11. CAP原理:
    1. C - Consistent ，一致性
    2. A - Availability ，可用性
    3. P - Partition tolerance ，分区容忍性
12. 因为 Codis 是无状态的，它只是一个转发代理中间件,用来管理Redis的集群.Redis-Cluster 也可以管理集群,并逐渐流行了起来.

## 拓展

1. Stream实现消息队列的方法以及相关的注意事项 以后用到详细的学习一下.
2. info相关的命令可以查询redis的状态.
3. 过期策略
   1. 过期的key会放入到字典表中,redis会定期的删除,还有一种处理是惰性删除,是零星的删除.
   2. 定时扫描策略:采用一种**贪心策略**,不会扫描字典中所有的key,而是随机选择20个key,删除这20个key中过期的key,如果过期的key比例超过1/4,就继续重负之前的步骤.
   3. 注意设置过期时间的时候加一个随机数,这样就不会导致大量的key一起失效了.
4. LRU算法:除了需要key/value字典外,还需要附加一个链表,链表中的元素最近访问的顺序进行排列.越不活跃的排在最后.
5. redis 使用了近似LRU算法.在每个key上增加了一个额外的24bit访问时间戳.LRU是懒惰处理,随机取出5个key,然后淘汰掉最旧的key,如果淘汰后内存还是超出 maxmemory，那就继续随机采样淘汰，直到内存低于maxmemory 为止。
6. 保护redis:所有不安全的指令例如删除库命令都可以更改命令.可以设置监听Ip,这样其它的ip发送的指令就会失效.

## 源码

1. Redis 的字符串叫着「SDS」，也就是 Simple Dynamic String。它的结构是一个带长度信息的字节数组。
2. Redis 的字符串有两种存储方式，在长度特别短时，使用 emb 形式存储 (embeded)，当长度超过 44 时，使用 raw 形式存储。Redis的对象头占16个字节.分配各头的空间的capacity+3,即19个字节.超出64个字节,redis就会判定为大对象,去掉一个/0结尾的分隔符,所以留给保存数据的长度的44字节.
   1. hash字典表如果存储数据的长度是一维数组的长度,那么需要扩容为2倍大小,但是当正在bgsave(快照)时候,就会延迟进行,除非空间不足,元素的个数高达一维数组长度的5倍.就会强制扩容.
3. 压缩列表:Redis 为了节约内存空间使用，zset 和 hash 容器对象在元素个数较少的时候，采用压
   缩列表 (ziplist) 进行存储.紧凑型没有冗余的空间,每次插入都会realoc扩展内存.
4. quickList:quicklist 是 ziplist 和 linkedlist 的混合体，它将 linkedlist 按段切分，每一段使用 ziplist 来紧凑存储，多个 ziplist 之间使用双向指针串接起来。默认单个ziplist长度为8k.
5. 跳跃链表没太看懂.

---

# Redis 设计与实现

## 第一部分 数据结构与对象

1. 简单动态字符串(SDS):
   1. <img src=".\picture\redis\SDS结构.png" style="zoom:80%;" />
   2. 空间预分配和惰性空间释放
2. 链表:当一个列表键包含的数据比较多的时候,Redis 就会使用链表作为列表键的底层实现.
3. 

