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
      # 或者
       docker exec -it redis-test /bin/bash
       ## 如果不成功
       #启动docker
      sudo systemctl start docker
      # 查看启动的镜像
      docker ps
      # 启动 redis
      docker start redis-test
      # 执行命令
      redis-cli
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

   3. Hash(字典):Redis 的字典相当于 Java 语言里面的 HashMap，它是无序字典。内部实现结构上同Java 的 HashMap 也是一致的，同样的数组 + 链表二维结构。不同的是rehash是**渐进式的**.k可以用来保存用户信息,避免像String 那样一次性将所有的字段取出来.

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
         > sadd books python # 重复
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
   
   6. hyperloglog:存入的数据量非常大,但是基数较少的数据.去重复,不直接存储数据本身,通过牺牲准确性来换取空间.
   
      1. ~~~shell
         PFADD jd 1 2 3  # 向键jd中添加数值 1 2 3
         PFCOUNT jd # 查询数量
         PFMERGE result jd1 jd2  #将jd1和jd2的数值相加赋值给result
         ~~~
   
      2. 应用场景
         1. 主要是统计日活数,将日期作为键,ip作为值,直接count获得答案.
   
   7. GEO:经纬度存储,底层使用zset实现.经纬度相当于zset的权值.
   
      1. ~~~shell
         GEOADD city 116.4003 39.96543 "天安门" #key为city的 经纬度 值 
         GEOPOS city 天安门                     # 返回城市天安门的经纬度.
         1) 116.4003 39.96543
         GEODIST city 天安门 故宫 m               #返回城市天安门和故宫的距离单位是米.
         GEORADIUS city 116.4003 39.96543 10 km withdist withcoord withhash count 10                                # 以给定的经纬度为原点,10km为半径 卸载距离,坐标,坐标的hash值 展示前10个.
         ~~~

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

5. 布隆过滤器:

   1. 布隆过滤器可以理解为一个不怎么精确的 set 结构.基本指令:bf.add 添加元素，bf.exists 查询元素是否存在.
   2. 如果不存在则一定不存在,如果存在则不能100%确定存在,因为存在Hash冲突,规避的办法是使用多个hash函数.

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
7. fork 多进程,redis做持久化的时候,会fork子进程,子进程刚刚产生时，它和父进程共享内存里面的代码段和数据段,进行持久化.而父进程有时候需要修改数据.进程这个时候就会使用操作系统的 **COW(Copy On Write) 机制**来进行数据段页面的分离。数据段是由很多操作系统的页面组合而成，当父进程对其中一个页面的数据进行修改时，会将被共享的页面复制一份分离出来，然后对这个复制的页面进行修改。这时子进程相应的页面是没有变化的，还是进程产生时那一瞬间的数据.
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
   2. 定时扫描策略:采用一种**贪心策略**,不会扫描字典中所有的key,而是随机选择20个key,删除这20个key中过期的key,如果过期的key比例超过1/4,就继续重复之前的步骤.
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

1. ### 简单动态字符串(SDS):
   
   1. <img src=".\picture\redis\SDS结构.png" style="zoom:80%;" />
   2. 与C语言字符串的比较:
      - 直接获取保存字符串的长度,不用计算.
      - 杜绝了缓存溢出同时避免了内存从新分配:因为不记录长度,当有新的字符添加进入的时候,如果使分配的空间不足.
      - 空间预分配:当有13个字节时候,实际分配给sds的长度为13+13+1=27个,1为结尾\0.
      - 惰性空间释放:当保存的字符串减小长度的时候,不会马上释放空间,而是等待下一次的使用.
      - 二进制安全:C语言的字符串只能保存文本,因为视频,图片其他类型转换为二进制会与字符串的结尾相混淆,无法识别出是二进制数据还是字符串的分割符.
   
2. ### 链表(List)

   1. 当一个列表键包含的数据比较多的时候,Redis 就会使用链表作为列表键的底层实现.
   2. 链表一般的结构是前置节点,后置节点和节点的值.

3. ### 字典(Hash)

   1. 由键值对组成  类似于java中的HashMap.

   2. ![](.\picture\redis\Hash字典表结构.png)

   3. 字段解释

       ![](.\picture\redis\Hash字典表字段.png)

   4. 空字典![](.\picture\redis\空字典.png)

   5. 字典表是数组和链表组成![](.\picture\redis\字典数组和链表.png)

   6. 字典表为了避免同时将所有数据进行rehash带来性能损失,采用的是**渐进式的rehash**,同时维护两个字典表,如果旧的没有就去新的里面找.

4. ### 跳跃表

   1. 实现有序集合键和集群节点中做内部数据结构.
   2. 由zskipList和zskipNode组成,zskipList保存了表头节点,表尾节点和长度.node表示跳跃表节点.
   3. 每个节点上不仅仅保存了数据对象,还保存了访问另外节点的指针,当访问数据的时候,不用按顺序进行遍历,而是根据指针在节点上快速的跳跃,花费很少的时间就能找到对象.
   4. 数据结构![](.\picture\redis\跳跃链表的遍历.png)
   
5. ### 整数集合

   1. 根据保存数据的类型来创建空间,如果空间大小进行升级或者降级.

6. ### 压缩列表

   1. 当一个列表键只包含少量列表项,并且每个列表项要么是小整型值,要么是比较短的字符串,Redis就会用压缩列表来保存.
   2. 组成结构:连续的内存块组成的顺序型数据结构.为了节约内存而开发.
   3. ![](.\picture\redis\压缩列表的组成部分.png)
   4. ![](.\picture\redis\压缩列表各部分说明.png)
   5. ![](.\picture\redis\压缩列表节点组成部分.png)
   6. 节点的previous_entry_length是上一个节点的长度,根据这个偏移量找到数据的起始位置.
   7. 由于压缩列表的结构紧凑,当更新数据和删除数据的时候,容易引发**连锁更新**,即一个节点更新内存长度,导致所有的节点都更新内存长度,但是由于保存的都是小数据即使发生也不会产生较大的性能损失.

7. ### 对象

   1. Redis的键都是字符串类型,值有String,list,Hash,Set,Zset五种类型.
   2. 字符串对象是唯一一种会被其他类型嵌套使用的对象.
   3. 列表list的底层数据结构有zipList和LinkedList两种,当满足一下条件时候,会自动使用zipList进行存储.
      - 列表对象保存的所有元素长度都小于64字节.
      - 元素数量小于512个.
   4. Hash对象底层数据结构是zipList或者hashtable两种.
   5. 集合对象(set)的编码可以是inset或者hashtable.
   6. 有序集合对象(Zset) 编码可以是zipList或者skipList.
   7. 类型检查和多态.通过类型检查确保命令可以正确的执行,多态是对不同的数据结构可以使用相同的命令操作.例如zipList和hashtable都可以LLEN命令,但是底层的方法是不一样的.
   8. 内存回收:使用**引用计数法**进行处理.

# 第二部分 单机数据库的实现

1. ### 数据库

   1. 默认创建16个数据库,select index 选择数据库.默认index为0.
   2. 数据库使用一个字典表记录了所有的键.包括过期的键.
   3. 过期键删除策略:定时删除,惰性删除,定期删除.redis采用惰性删除和定期删除两种策略.
   4. RDB会所键过期的检查,过期的键不会做持久化或者数据的恢复.AOF也会做键过期检查,会在日志中增加一个delete删除键的操作.
   5. 复制 从服务器不会进行过期检查,不处理过期的键.

2. ### RDB持久化

   1. SAVE持久化会阻塞进程,BGSAVE可以派生一个子进程.

   2. AOF更新的频率比RDB高,服务器优先使用AOF还原数据.

   3. SAVE可以设置为保存条件.例如900s内对数据进行了一次修改.

   4. 数据结构

      ![](.\picture\redis\RDB数据的大结构.png)![](.\picture\redis\RDB文件的数据结构.png)

   5. 文件结构代表的意思:

      - REDIS:字符,程序载入文件时,快速检查是否为RDB文件
      - db_version:记录了RDB的版本号.
      - database:数据结构部分.
        - SELECTDB:提示程序接下来是数据库版本号.
        - 0 数据库版本号
        - pairs:所有键值对 由 EXPIRETIME(有过期时间的键才有) type key value 三或者四部分组成.
      - EOF:结束标志
      - check_sum:参数校验和.

3. ### AOF持久化

   1. 将客户端所执行的命令保存在日志中.
   2. AOF重写:为了避免AOF过度膨胀,提供了日志重写的功能,使多条语句命令合并为一个语句.
   3. AOF重写缓冲区:用来保证当AOF重写时候,有新的命令执行.会定期的进行重写.

4. RDB与AOF的比较

   1. RDB:RDB是每隔一段时间自动将内存中的数据集快照写入磁盘中,也就是我们所说的Snapshot快照,他恢复(读取)数据是将快照文件直接读取到内存中.
   2. AOF:AOF是用日志的形式将每一个写操作都记录在日志文件中,只允许进行增量操作,不允许进行修改操作.他恢复(读取)数据来执行过的操作重新执行一遍.
   3. 所以RDB恢复数据块,但是肯能导致数据没来得及快照而导致数据的丢失.AOF由于记录每一次的增量操作,数据记录完整,但是文件较大,需要有重写技术,恢复数据的时间比较慢.

5. ### 事件

   1. **文件事件:**Redis服务器通过套接字与客户端进行连接,而文件事件就是服务器对套接字操作的抽象.
   2. 把多个套接字放在队列里面,队列满时候在进行命令的发送,而实现**多路复用**的目的.
   3. 如果一个套接字又可读又可写的时候,服务器会先执行读的操作.
   4. 文件事件的处理器
      - 连接应答处理器
      - 命令请求处理器
      - 命令回复处理器
   5. **时间事件**:将所有的定时任务放在无序链表中,每当时间执行器执行的时候,就会遍历列表.无序是指没有按照时间进行排序.

## 第三部分

1. ### 复制

   1. 旧版本的同步功能通过同步(sync)和命令传播(command propagete)两个操作:
      - 同步:是将从服务器更新到到主服务器相同的数据库状态.
      - 命令传播:主服务器的状态被修改,使主从恢复到一致的状态.
   2. 同步:通过sync命令来实现,主服务器生成一个RDB文件,从服务器读取进行同步.
   3. 命令传播:当执行同步的时候,主服务器会有命令并行的执行,导致主从服务器数据不一致,命令传播会就是将主服务器的刚刚执行的命令发送给从服务器执行.
   4. 旧版本同步功能的缺陷:当服务器断线恢复后,需要将整个RDB文件从新执行,尤其不同步的数据仅仅有几条的,完全没有必要.
   5. 从2.8以后的新版本,用psync命令取代了sync命令.有完成重同步和部分重同步两种模式.
   6. 部分重同步的实现
      - 主服务器的复制偏移量和从服务器的复制偏移量.
      - 主服务器的复制积压缓冲区
      - 服务器的运行ID.
   7. **复制偏移量**:主服务器向从服务器发送N个字节的时候,就会在自己的**偏移量**上加N.从服务器每次接收的时候也会加N.会根据这个偏移量来判断是否与主服务器进行了同步.
   8. **复制积压缓冲区**:由一个先进先出队列组成.当主服务器发送写命令给从服务器时候,也会将**命令和偏移量**放进这个队列里面.当部分同步的时候,会根据偏移量执行命令.
   9. **服务器运行ID**:执行同步的时候会复制这个id,如果主从不一致,说明没有同步成功.

2. ### 集群模式

   1. 节点:集群有多个节点组成.使用命令 CLUSTER MEET <ip> <port> 命令 进行握手,加入到集群.
   2. 集群的数据结构:clusterNode包含了节点的信息,其中保存连接节点所有的有关信息clusterLink ;clusterLink中又包含了连接节点的信息clusterState在当前节点的视角下,记录了集群目前所处的状态,集群是在线还是下线,集群包含多少个节点,集群当前的配置纪元.包括集群节点名单.
   3. ![](.\picture\redis\clusterState数据结构.png)
   4. **槽指派**:一共有16383个槽,通过命令 CLUSTER ADDSLOTS <slot> [slot ....]指派.节点会将自己管理的槽广播出去,每个节点都会知道其他节点负责管理的槽.将他们保存在一个数组中.
   5. **命令执行**:对键进行hash,计算出它对应的槽,如果是是当前节点管理的槽,就直接执行,如果不是,就发送**moved错误**给客户端,根据返回的ip和端口号指引客户端将命令转移到对应的节点进行处理.
   6. **ASK错误**:当从新分配槽节点的时候,可能会出现一部分数据在目标的节点,一部分在源节点上,当数据在源节点没有找到的时候,就会发送ASK的错误给客户端.
   7. **ASKING命令**:当客户端携带ASKING标识的时候,会破例执行所对应的命令一次,而不是返回给客户端moved错误.????
   8. **MOVED错误和ASK错误的区别**:MOVED错误表示槽的负责权已经从一个节点转移到了另一个节点,而ASK错误只是两个节点在迁移槽的过程中使用的一种临时的措施.
   9. **复制与故障转移**:半数以上的主节点认为掉线了,就会在给节点上打上fail下线标签.
10. 选举新的主节点:当有主节点挂掉的时候,会通知其他主节点对该节点的从节点进行投票,重新选取主节点.
## 第四部分 独立功能的实现

1. ### 发布与订阅

   1. 列表保存了所有的频道,频道对到又对应着客户端的链表,保存了所有订阅此频道的客户端.

2. ### 事务

   1. 通过MULTI,EXEC,WATCH命令来实现事务.
   2. 事务的实现
      - 事务开始:multi命令 打开事务的标识
      - 命令入队:EXEC,DISCARD,WATCH,MULTI四个命令的其中一种,会立即执行,其余命名会放入到队列中去.
      - 事务的执行:先进先出的队列中拿去命令执行.
   3. Watch命令的实现
      1. 是一种乐观锁,它可以在exec命令执行之前,监视任意数量的数据库键,并在Exec命令执行时,检查被监视的键是否至少被修改过了,如果是拒绝执行该事务,并向客户端返回事务失败的回复.
      2. 如果有修改,表示会被打开,事务执行的时候就会发现.
   4. ACID事务的原子性,一致性,隔离性,持久性.

3. ### 排序

   1. ~~~shell
      ## 对numbers list进行排序
      rpush numbers 5 3 1 2 4
      sort numbers
      ## 对字符串进行排序  ALPHA 
      sadd alphabet a c d b
      sort alphabet ALPHA
      ##按元素分值排序
      sadd test-result 3.0 jack 2.0 tom 1.0 lihua
      srange test-result 0 -1
      ## 为各元素设置序号
      mset Peter_number 1 tom_number 2 jack_number 3
      Sort test-result by *_number
      ~~~
      
   2. 不论是哪种排序,都是将元素先放入到数组中,进行排序.
   
   3. 排序可以使用关键字 limit.
   
   4. 排序后需要取出特定值 用get
   
      1. ~~~shell
         sort alphabet ALPHA get *_a
         ~~~
   
   5. store:由于sort只对结果进行排序,而不会保存结果,所以可以使用store把结果保存在指定的键中.相当于对结果执行RPUSH命名.
   
   6. 排序各字段执行的顺序:
   
      - 排序:命令使用ALPHA,ASC,DESC,By选项时候,进行排序.
      - LIMIT
      - 获取外部键 GET命令.
      - 保存排序结果 store
      - 结果返回
   
   7. 进行SORT命名时候,除了Get命令之外,尽是改变选项的顺序并不会影响执行的结果.
   
4. ### 二进制位数组

   1. bit 常用命令

      ~~~shell
      ## 在0的位置设置的为1
      SETBIT bit 0 1
      ## 获取第三位的bit值 (0 或者 1)
      GETBIT bit 3
      ## 有多少位1 
      BITCOUNT bit
      ## 按位与
      BITOP AND and-result x y z
      ##按位或
      BITOP OR or-result x y z
      ## 按位 异 或
      BITOP XOR xor-result x y z
      ## 取反 not
      BITOP NOT not-value value
      ~~~

   2. 二进制位统计算法(BITCOUNT命令):计算汉明重量和查表和.

5. ### 慢查询日志

   1. 两个重要的参数
      1. slowlog-log-slower-than:指定执行时间超过多少微秒的命令会被记录到查询的日志中.
      2. slowlog-max-len:最多保存多少条慢查询日志.

6. ### 监视器

   1. Monitor  命名 让一个客户端变成一个监视器 监视所有执行的命令.

---

# Redis 知识更新

## 基础

### redis是单线程还是多线程

1. redis是单线程的,但是在4.0版本之后引入了多线程,这是因为在删除大key的时候,回到阻塞,使用unlink命令,是懒删除.
2. 为什么用单线程
   1. 基于内存操作
   2. 简单的数据结构.
   3. 多路复用和非阻塞I/O:多路复用功能来监听多个Socket客户端.
   4. 避免上下文的切换. 线程的上下文切换.
3. 是单线程到遇到CPU的多线程怎么处理呢
   1. Redis操作在内存,瓶颈是内存大小或者网络,而不会是CPU.
4. 多个I/O线程解决网络IO问题.单个线程负责处理具体的任务.

## 缓存穿透

1. 使用空对象和bloomFilter解决.
   1. 空对象 当有请求到redis中,返回一个空对象.但是对于恶意攻击,每次都有一个新的key,不能有效的解决.
   2. 通过bloomFilter过滤器判断是否存在key,如果没有直接返回空值,如果有就请求redis,如果这次不是误判,从redis取值返回,如果不存在则在请求mysql返回. 

## 缓存击穿

1. 热点key失效,
   1. 定时轮询,互斥更新(先更新A在更新B,查询的时候先查询B,在查询A.),差异失效时间.
   2. 对于热点数据 不设置过期时间.
   3. 互斥独占锁防止击穿,加锁前查询一次,加锁后在查询一次.

## 分布式锁

### 解决的一般步骤以及存在的问题

1. 为了避免服务宕机导致锁不能释放,所以需要设置锁过期时间,自动使锁失效.
2. 设置过期时间可能会导致锁被误删了.解决办法是判断这个锁是自己的才进行释放.
3. 由于判断不是原子性的 所以需要lua脚本保证原子性.
4. 超时时间不够用了 锁释放时间的续约问题.
5. 最后版本通过 Redisson 实现分布式锁.

### 详细文档讲解

#### 为什么需要分布式锁

​	我们知道，当多个线程并发操作某个对象时，可以通过synchronized来保证同一时刻只能有一个线程获取到对象锁进而处理synchronized关键字修饰的代码块或方法。既然已经有了synchronized锁，为什么这里又要引入分布式锁呢？

​	因为现在的系统基本都是分布式部署的，一个应用会被部署到多台服务器上，synchronized只能控制当前服务器自身的线程安全，并不能跨服务器控制并发安全。比如下图，同一时刻有4个线程新增同一件商品，其中两个线程由服务器A处理，另外两个线程由服务器B处理，那么最后的结果就是两台服务器各执行了一次新增动作。这显然不符合预期。

![img](.\\picture\redis\多线程.png)

​	而本篇文章要介绍的分布式锁就是为了解决这种问题的。

#### 什么是分布式锁

​	分布式锁，就是控制分布式系统中不同进程共同访问同一共享资源的一种锁的实现。

​	所谓当局者迷，旁观者清，先举个生活中的例子，就拿高铁举例，每辆高铁都有自己的运行路线，但这些路线可能会与其他高铁的路线重叠，如果只让高铁内部的司机操控路线，那就可能出现撞车事故，因为司机不知道其他高铁的运行路线是什么。所以，中控室就发挥作用了，中控室会监控每辆高铁，高铁在什么时间走什么样的路线全部由中控室指挥。

​	分布式锁就是基于这种思想实现的，它需要在我们分布式应用的外面使用一个第三方组件（可以是数据库、Redis、Zookeeper等）进行全局锁的监控，由这个组件决定什么时候加锁，什么时候释放锁。

![img](.\\picture\redis\分布式锁.png)

#### Redis如何实现分布式锁

​	在聊Redis如何实现分布式锁之前，我们要先聊一下redis的一个命令：setnx key value。我们知道，Redis设置一个key最常用的命令是：set key value，该命令不管key是否存在，都会将key的值设置成value，并返回成功：

![img](.\\picture\redis\redis赋值1.png)

​	setnx key value 也是设置key的值为value，不过，它会先判断key是否已经存在，如果key不存在，那么就设置key的值为value，并返回1；如果key已经存在，则不更新key的值，直接返回0：

![img](.\\picture\redis\redis赋值2.png)

● **最简单的版本：setnx key value**

​	基于setnx命令的特性，我们就可以实现一个最简单的分布式锁了。我们通过向Redis发送 setnx  命令，然后判断Redis返回的结果是否为1，结果是1就表示setnx成功了，那本次就获得锁了，可以继续执行业务逻辑；如果结果是0，则表示setnx失败了，那本次就没有获取到锁，可以通过循环的方式一直尝试获取锁，直至其他客户端释放了锁（delete掉key）后，就可以正常执行setnx命令获取到锁。流程如下：

![img](.\\picture\redis\最简单版本.png)

​	这种方式虽然实现了分布式锁的功能，但有一个很明显的问题：没有给key设置过期时间，万一程序在发送delete命令释放锁之前宕机了，那么这个key就会永久的存储在Redis中了，其他客户端也永远获取不到这把锁了。

**● 升级版本：设置key的过期时间**

​	针对上面的问题，我们可以基于setnx key value的基础上，同时给key设置一个过期时间。Redis已经提供了这样的命令：set key value ex seconds  nx。其中，ex seconds 表示给key设置过期时间，单位为秒，nx 表示该set命令具备setnx的特性。效果如下：

![img](.\\picture\redis\redis赋值过期时间1.png)

​	我们设置name的过期时间为60秒，60秒内执行该set命令时，会直接返回nil。60秒后，我们再执行set命令，可以执行成功，效果如下：

![img](.\\picture\redis\redis赋值过期时间2.png)

​	基于这个特性，升级后的分布式锁流程如下：

![img](.\\picture\redis\最简单版本2.png)

​	这种方式虽然解决了一些问题，但却引来了另外一个问题：存在锁误删的情况，也就是把别人加的锁释放了。例如，client1获得锁之后开始执行业务处理，但业务处理耗时较长，超过了锁的过期时间，导致业务处理还没结束时，锁却过期自动删除了（相当于属于client1的锁被释放了），此时，client2就会获取到这把锁，然后执行自己的业务处理，也就在此时，client1的业务处理结束了，然后向Redis发送了delete  key的命令来释放锁，Redis接收到命令后，就直接将key删掉了，但此时这个key是属于client2的，所以，相当于client1把client2的锁给释放掉了：

![img](.\\picture\redis\最简单版本3.png)

**● 二次升级版本：value使用唯一值，删除锁时判断value是否当前线程的**

​	要解决上面的问题，最省事的做法就是把锁的过期时间设置长一点，要远大于业务处理时间，但这样就会严重影响系统的性能，假如一台服务器在释放锁之前宕机了，而锁的超时时间设置了一个小时，那么在这一个小时内，其他线程访问这个服务时就一直阻塞在那里。所以，一般不推荐使用这种方式。

​	另一种解决方法就是在set key value ex seconds nx时，把value设置成一个唯一值，每个线程的value都不一样，在删除key之前，先通过get  key命令得到value，然后判断value是否是自己线程生成的，如果是，则删除掉key释放锁，如果不是，则不删除key。正常流程如下：

![img](.\\picture\redis\升级版本1.png)

​	当业务处理还没结束的时候，key自动过期了，也可以正常释放自己的锁，不影响其他线程：

![img](.\\picture\redis\升级版本2.png)

​	二次升级后的方案看起来似乎已经没什么问题了，但其实不然。仔细分析流程后我们发现，判断锁是否属于当前线程和释放锁两个步骤并不是原子操作。正常来说，如果线程1通过get操作从Redis中得到的value是123，那么就会执行删除锁的操作，但假如在执行删除锁的动作之前，系统卡顿了几秒钟，恰好在这几秒钟内，key自动过期了，线程2就顺利获取到锁开始执行自己的逻辑了，此时，线程1卡顿恢复了，开始继续执行删除锁的动作，那么此时删除的还是线程2的锁。

![img](.\\picture\redis\升级版本3.png)

**● 终极版本：Lua脚本**

​	针对上述Redis原始命令无法满足部分业务原子性操作的问题，Redis提供了Lua脚本的支持。Lua脚本是一种轻量小巧的脚本语言，它支持原子性操作，Redis会将整个Lua脚本作为一个整体执行，中间不会被其他请求插入，因此Redis执行Lua脚本是一个原子操作。

​	在上面的流程中，我们把get key value、判断value是否属于当前线程、删除锁这三步写到Lua脚本中，使它们变成一个整体交个Redis执行，改造后流程如下：

![img](.\\picture\redis\最终版1.png)

​	这样改造之后，就解决了释放锁时取值、判断值、删除锁等多个步骤无法保证原子操作的问题了。关于Lua脚本的语法可以自行学习，并不复杂，很简单，这里就不做过多讲述。

​	既然Lua脚本可以在释放锁时使用，那肯定也能在加锁时使用，而且一般情况下，推荐使用Lua脚本，因为在使用上面set key value ex seconds  nx命令加锁时，并不能做到重入锁的效果，也就是当一个线程获取到锁后，在没有释放这把锁之前，当前线程自己也无法再获得这把锁，这显然会影响系统的性能。使用Lua脚本就可以解决这个问题，我们可以在Lua脚本中先判断锁（key）是否存在，如果存在则再判断持有这把锁的线程是否是当前线程，如果不是则加锁失败，否则当前线程再次持有这把锁，并把锁的重入次数+1。在释放锁时，也是先判断持有锁的线程是否是当前线程，如果是则将锁的重入次数-1，直至重入次数减至0，即可删除该锁（key）。

![img](.\\picture\redis\最终版2.png)

​	实际项目开发中，其实基本不用自己写上面这些分布式锁的实现逻辑，而是使用一些很成熟的第三方工具，当下比较流行的就是Redisson，它既提供了Redis的基本命令的封装，也提供了Redis分布式锁的封装，使用非常简单，只需直接调用相应方法即可。但工具虽然好用，底层原理还是要理解的，这就是本篇文章的目的。

### Redisson

1. 主从复制,获取锁后 主节点挂掉了,从节点设计为主节点,还能获取这个锁.
2. 源码
   1. 默认缓存延续的时间是30s.
   2. watch dog 每隔10s中检查一次,延长30s时间.
   3. ![](.\picture\redis\redisson可重入锁.png)

### IO多路复用

1. redis利用epoll来实现多路复用,讲连接信息和事件放入到队列中,一次放到文件事件分派器,分派器将事件发送给处理器.
2. Select 函数是讲连接集合遍历放入到内核中,这样避免了内核态会用户态的切换.使用类似于bitmap,讲有数据需要链接的位变为1. 
   1. 缺点
      1. bitmap 最大1024位,一个进程最多能处理1024个连接
      2. &rset不可重用
      3. 连接的数组拷贝到了内核态,有开销.
      4. 并没有通知y用户哪一个有socket数据,仍然需要遍历.
3. Poll 函数:遍历查找fd,并为了复用重置状态.解决select的bitmap 1024个连接和不能复用的
4. EPoll函数:直接在内核态操作,底层使用的是红黑数结构,将需要处理的进行进行通知回调,避免了遍历.

### 淘汰策略or回收策略

1. maxmemory设置最大使用内存空间.

2. 对于所有键值对有三种淘汰策略一种你拒绝策略

   1. LRU:最近最少使用算法.Redis 中，LRU算法被做了简化，以减轻数据淘汰对缓存性能的影响。Redis 默认会记录每个数据的最近一次访问的时间戳（由键值对数据结构RedisObject 中的 lru 字段记录）。

      1. Redis在决定淘汰的数据时，第一次会随机选出N 个数据，把它们作为一个候选集合。
      2. Redis 会比较这 N 个数据的 lru 字段，把lru 字段值最小的数据从缓存中淘汰出去。
      3. 当需要再次淘汰数据时，Redis 需要挑选数据进入第一次淘汰时创建的候选集合。挑选标准是：
         - 能进入候选集合的数据的 lru 字段值必须小于候选集合中最小的 lru 值。
         - 有新数据进入候选数据集后，如果候选数据集中的数据个数达到了 maxmemory-samples，Redis 就把候选数据集中 lru 字段值最小的数据淘汰出去。
      4. Redis 提供了一个配置参数 maxmemory-samples，这个参数就是 Redis 选出的数据个数N。例如，执行如下命令，可以让 Redis 选出 100 个数据作为候选数据集：

      ```shell
      CONFIG SET maxmemory-samples 100
      ```

   2. LFU（least frequently used (LFU) page-replacement algorithm）。即最不经常使用页置换算法

      1. 一是，数据访问的时效性（访问时间离当前时间的远近）；
      2. 二是，数据的被访问次数。
      3. LFU 缓存策略是在 LRU 策略基础上，为每个数据增加了一个计数器，来统计这个数据的访问次数。当使用 LFU 策略筛选淘汰数据时：
         1. 首先会根据数据的**访问次数**进行筛选，把访问次数最低的数据淘汰出缓存。
         2. 如果两个数据的访问次数相同，LFU 策略再比较这两个数据的访问时效性，把距离上一次访问时间更久的数据淘汰出缓存。
         3. 如果一个key的计数器值特别大，但是长时间没有被访问，redis中LFU算法会有对应的衰减机制
                lfu_log_factor衰减因子：控制计数器值增加的速度，避免 counter（8bit） 值很快就到 255 了。
                lfu_decay_time默认1：如果数据在 N 分钟内没有被访问，那么它的访问次数就减 N。

   3. 随机选择:随机选择并删除数据.

   4. noevction:内存满了Redis拒绝服务.

3. 针对设置过期时间的键值对，会自动过去，但是如果还没到自动过期而内存占满，则也会采用淘汰机制。除了适用于所有键的策略外，还有

   1. ttl：从设置了过期时间的键值对中，根据`过期时间的先后进行删除，越早过期的越先被删除`.
