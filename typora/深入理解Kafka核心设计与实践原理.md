# 深入理解Kafka核心设计与实践原理

1. ~~~shell
   ## 启动kafka命令
   ./bin/kafka-server-start.sh  config/server.properties
   ~~~

2. ~~~shell
   ##启动docker
   systemctl start docker
   ~~~

3. ~~~shell
   ##用docker启动zookeeper
   sudo docker run --name zookeeper2 -p 2181:2181  --restart always -d zookeeper:3.4.12
   ~~~

4. 使用docker安装kafka

   ~~~shell
   ## 进入容器中
   docker exec -it kafka /bin/sh
   ## kafka对应的位置
   cd /opt/kafka_2.11-2.0.0/bin
   ~~~

5. 消费者参数:

   - AR集合:分区中所有副本的统称为AR.   AR=ISR+OSR.
   - ISR集合:所有与leader保持一定同步的副本组成.
   - OSR集合:与leader副本同步滞后过多的副本组成OSR(out-of-Replica)

6. Key会被kafka server用来做为将数据分区的参数.

7. 消费完数据也需要提交commit,来更新队列里面处理消息的位置(下一次拉取的消息的位置).一般都是手动提交.

### 主题与分区

1. 查看主题

   - ~~~shell
     #查看所有可用的主题
     ./bin/kafka-topics.sh --zookeeper localhost:2181/kafka -list
     ~~~

   - ~~~shell
     #--topic 指定多个主题 显示主题的信息
     ./bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic demo 
     ~~~

   - ~~~shell
     # under-replicated-partitions找出所有包含失效副本的分区
     ./bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic demo --under-replicated-partitions
     ~~~

   - ~~~shell
     # --unavailable-partitions 查看主题中没有leader副本的分区
     ./kafka-topics.sh --zookeeper localhost:2181 --describe --topic topic-demo --unavailable-partitions
     ~~~
   
2. 创建主题 test 3个副本 2个分区

      ~~~shell
   ./kafka-topics.sh --zookeeper localhost:2181 --create --topic test --replication-factor 3 --partitions 2
   ~~~

3. 修改分区

   - ~~~shell
     #分区数修改为3
     ./kafka-topics.sh --zookeeper localhost:2181/kafka --alter --topic demo --partitions 3  5
     ~~~

4. 分区数量与linux系统的文件描述符有关,一般上线是硬限制描述符4096个,可以手动设置增大.

5. 控制台模拟消费端

   - ~~~shell
     ## 从开始消费
     ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic flow_log --from-beginning
     ## 指定消费组
     ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic flow-event-log --consumer-property group.id=flowgroup2
     ##或者
     ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic flow-event-log --from-beginning --group flowgroup1
     ~~~
     
   - ~~~shell
     ## 模拟生产者
     ./kafka-console-producer.sh --broker-list localhost:9092 --topic flow_log
     ~~~

6. 查看消费组消费情况

   - ~~~shell
     ## 查看所有消费组
     ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
     ## 查看指定消费组
     ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group flowgroup2
     ## 删除消费组
     ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --delete --group flowgroup3  
     ## 查看消费组状态
      ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group flowgroup4 --state
      ## 查看消费组内消费者的信息以及分配情况
     ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group flowgroup4 --members --verbose
     ~~~

### 注意:有时候kafka在server.properties的文件中配置连接的路径是localhost:2181而不是 localhost:2181/kafka 所以执行命令不能带/kafka.

### 注意2:消息的位移是递增的,而且保存消息的日志是有效期默认为7天,过了时间后,即使看到了位移,也不能消费到之前的数据了.

------

## 第一章 初始kafka

1. 主题与分区:Kafka中消息以主题为单位进行归类,生产者和消费者要与主题进行交互.一个主题可以细分为多个分区,分区在存储层面可以看做一个可追加的日志.消息被追加到分区日志文件时,都会分配一个特定的偏移量.Kafka保证的是分区有序而不是主题有序.
2. 分区中有主副本(leader)和从副本(follow)组成(一主多从).
   - 消息会先发送到主副本,然后从副本拉取主副本上的消息进行同步.leader副本只负责读写的请求,从副本只负责同步主副本的消息,等到主副本挂掉的时候,重新选举一个主副本.
   - 从副本会同步消息会有一定的滞后,当**所有的从副本**都同步了某条消息后,这条消息才会被消费者消费到.每个副本都维护一个offset,所有的副本都具有这个offset,将这个offset成为**高水位**.
3. Kafka复制机制:即不是完全的同步复制,也不是单纯的异步复制.完全同步复制,所有能工作的follow副本都同步完,才能被消费,影响了性能.完全的异步复制,如果主节点挂掉会出现消息的丢失.

## 第二章 生产者

1. 发送消息有三种模式:发后即忘,同步,异步.

2. Kafka消息发送send本来就是异步的,可以在返回值调用get来阻塞,等待返回值,变成同步的请求.

   1. ~~~java
      Future<RecordMetadata> future = producer.send(record);
      // 阻塞 变成同步的请求.
      RecordMetadata metadata = future.get();
      System.out.println(metadata.topic() + "-" +
      metadata.partition() + ":" + metadata.offset());
      ~~~

   2. 异步发送 调用一个回调函数 ,可以**保证分区有序性.**

      ~~~java
      // CallBack 回调函数
      producer.send(record, new Callback() (
      @Override
      public void onCompletion(RecordMetadata metadata, Exception exception) (
      if (exception ! = null) (
      exception.printStackTrace();
      } else (
      System.out.println(metadata.topic() + "-" +
      metadata.part1tion() + " : " + metadata.offset());}}
      }) ;
      ~~~

   3. produce.close();会等待所有的请求执行发送完之后,在调用关闭方法.也可以在close设置等待的时间,进行强行的关闭.

   4. 可以自定义序列化格式,需要实现Serializer接口.

      1. ~~~java
         public class CompanySerializer implements Serializer<Company> 
         ~~~

   5. 如果发送消息的时候指定了分区,需要用到分区器.
   
   6. 生产者客户端的整体架构
   
      1. ![](..\typora\picture\kafka\生产者客户端的整体架构.png)
      2. 由主线程和Sender线程组成,主线程会将消息保存到**消息累加器中**,累加器的大小可以通过buffer.memory来配置,不同的分区中底层的数据结构是多个双端队列.通过设置batch.size参数设定producerBatch的大小.
      3. Sender从RecordAccumulator获取缓存消息后,会进一步将<分区,Deque<ProducerBatch>>的保存形式转换为<Node,List<ProduderBatch>>的形式.
      4. inFlightRequest通过未回应的请求判断Node节点中负载最小的那一个.
      5. 记录集群的节点,分区信息的都称为元数据.

## 第三章 消费者

1. 消息投递的模式:**点对点模式和发布订阅模式**.当所有的消费者都隶属于同一个消费组,消息均衡的投递给每一个消费者,这时使用点对点模式.

2. subscribe()和assign()都能订阅消息,但subscribe具有消费者自动再均衡的功能.

3. 消费模式有推模式和拉模式两种.

4. ConsumerRecords(TopicPartition)方法来获取消息集中指定分区的消息.

   1. ```java
      ConsumerRecords<String, String> records=consumer.poll(Duration.ofMillis(lOOO));
      for (TopicPartition tp : records.partitions()) {
      for (ConsumerRecord<String, String> record : records.records(tp)) {
      System.out.println(record.partition()+" : "+record.value());}}
      ```

5. 按照主题进行消费的方法.

   1. ~~~java
      public Iterable<ConsumerRecord<K, V>> records(String topic)
      ~~~

6. 位移的提交动作在还未成功消费之前提交了,会出现**消息丢失.**当消费到X+5出现故障恢复后,重新拉取消息,x+2到x+4之间的消息又消费了一遍.出现了**重复消费**的问题.

7. 在Kafka中消费者找不到所记录的消费位移时,就会根据消费者客户端参数**auto.offset.reset**配置决定从何处开始消费.latest表示从分区末尾开始消费.earliest表示从0开始消费.none找不到就抛出异常.

8. seek()有指定分区位置消费的方法,还可以指定时间,消费到昨天或者之前的数据.

9. 在均衡是指分区的所属权从一个消费者转移到另一个消费者的行为.

   1. ~~~java
      consumer.subscribe(Arrays.asList(topic), new ConsumerRebalanceListener () {
      @Override
      public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
      consumer.commitSync(currentOffsets);
      currentOffsets.clear();
      @Override
      public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
      //do nothing.
      }
      }) ;
      ~~~

   2. 调用ConsumerRebalanceListener,重写方法,在均衡器触发之前和之后进行相应的操作.

10. 消费者拦截器:在poll消息抵达之前进行一些操作.

11. 多线程实现：

    1. 常用的方法是**线程封闭**,每个线程实例化一个KafkaConsumer对象。一个线程可以消费一个或者多个分区中的消息，所有的消费线程隶属于同一个消费组。这种实现方式的并发度受限于分区的实际个数。
    2. 多个消费线程消费同一个分区。由于对位移提交和顺序控制处理变得非常复杂，实际应用中使用的极少。不推荐。

12. 多线程代码的实现

    1. ~~~java
       public class Consumer {
           public static final String brokerList = "10.7.215.130:9092";
       
           public static final String groupid = "group1";
           public String topic = "topic-demo";
       
           public static Properties initConfig() {
               Properties props = new Properties();
               props.put("bootstrap.servers", brokerList);
               props.put("group.id", groupid);
               props.put("enable.auto.commit", "true");
               props.put("auto.commit.interval.ms", "1000");
               props.put("session.timeout.ms", "30000");
               props.put("max.poll.records", 1000);
               props.put("auto.offset.reset", "earliest");
               props.put("key.deserializer", StringDeserializer.class.getName());
               props.put("value.deserializer", StringDeserializer.class.getName());
               return props;
           }
       
           public Consumer() {
               Properties props = initConfig();
               int consumerThreadNum = 3;
               for (int i = 0; i < consumerThreadNum; i++) {
                   new KafkaConsumerThread(props).start();
               }
           }
       
           public static void main(String[] args) {
               new Consumer();
           }
       }
       ~~~

    2. ~~~java
       public class KafkaConsumerThread extends Thread {
       
           private AtomicBoolean runBoolean=new AtomicBoolean(true);
       
           private KafkaConsumer<String, String> kafkaConsumer;
       
           public String topic = "topic-demo";
       
           public KafkaConsumerThread(Properties props) {
               kafkaConsumer = new KafkaConsumer<>(props);
               kafkaConsumer.subscribe(Arrays.asList(topic));
           }
       
           @Override
           public void run() {
               try {
                   while (runBoolean.get()) {
                       ConsumerRecords<String, String> records =
                               kafkaConsumer.poll(Duration.ofMillis(1000));
                       for (ConsumerRecord<String, String> record : records) {
                           // 处理消息模块
                           System.out.println("record key is=" + record.key() + "==================value is " + record.value());
                       }
       //                runBoolean.set(false);
                   }
               } catch (Exception e) {
                   e.printStackTrace();
               } finally {
                   kafkaConsumer.close();
               }
           }
       }
       ~~~

    3. ~~~java
       // 版本 二
       // 消费消息耗时的步骤在一般在消息的处理上,所以尽量使用多线程
       public class KafkaConsumerThread extends Thread {
       
           private AtomicBoolean runBoolean=new AtomicBoolean(true);
       
           private KafkaConsumer<String, String> kafkaConsumer;
       
           public String topic = "topic-demo";
       
           public KafkaConsumerThread(Properties props) {
               kafkaConsumer = new KafkaConsumer<>(props);
               kafkaConsumer.subscribe(Arrays.asList(topic));
           }
       
           @Override
           public void run() {
               try {
                   while (runBoolean.get()) {
                       ConsumerRecords<String, String> records =
                               kafkaConsumer.poll(Duration.ofMillis(1000));
                       // 处理消息模块
                       ExecutorService executorService  =  new ThreadPoolExecutor(3, 3,
                               0L, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(1000),
                               new ThreadPoolExecutor.CallerRunsPolicy());
                       executorService.submit(new RecordHandler(records));
                   }
               } catch (Exception e) {
                   e.printStackTrace();
               } finally {
                   kafkaConsumer.close();
               }
           }
       }
       
       
       // 另一个新的类 避免看上去很乱 写在一起
       public class RecordHandler extends Thread {
       
           private ConsumerRecords<String, String> records;
       
           public RecordHandler(ConsumerRecords<String, String> records) {
               this.records = records;
           }
       
           @Override
           public void run() {
               for (ConsumerRecord<String, String> record : this.records) {
                   System.out.println("record key is=" + record.key() + "==================value is " + record.value());
               }
           }
       }
       ~~~

    4. 滑动窗口的方案以后再研究一下.

## 第四章 主题与分区

1. 分区可以有一至多个副本,每个副本对应一个日志文件,每一日志文件对应一至多个日志分段,每个日志分段还可以细分为索引文件,日志存储文件,快照文件等.

2. 当我们创建多个分区数为4,负载因子为2,一共8个日志文件,当有3个broker(服务器3个),kafka会均分为2,3,3.如果有9个,会均分为3,3,3.除了用kafka-topic.sh脚本自动分配,还可以通过**replica-assignment**手动分配方案.

   1. 

      ~~~shell
       bin/kafka  topics.sh  --zookeeper  localhost:2181/
      kafka  -- create -- topic topic-create-same --replica-assignment 2:0,0:1,1:2,2:1
      ~~~

   2. 2:0 分级代表分区号为2,0负载因子号为0.

   3. 如果指定了重复的副本,会报AdminCommandFailedExecption.

3. kafka在内部做埋点时会根据主题的名称来命名metric的名称,会将"."改写成"_",所以有时候会出现创建主题名称错误的异常.

4. broker.rack=RACK1,指定了机架上的名称,如果集群中仅有部分指定了机架名称,回报AdminOperationException异常.

5. 分区的计算规则 -----之后在研究

6. under-replicated-partitions和unavailable-partpartitions都可以找到有问题的分区.unavailable-partpartitions参数可以找出所有包含失效副本的分区.

   1. ~~~shell
      bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic topic-create --under-replicated-partitions
      ~~~

7. kafka支持增加分区数,而不支持减少分区数.

8. ~~~shell
   # 查看所有主题的配置参数
   ./kafka-configs.sh --zookeeper localhost:2181 --describe --entity-type topics
   ~~~

9. kafkaAdminClient通过它可以在代码中操作主题等.

10. 优先副本选举:可以通过path-to-json-file定义个json文件,当选举发生的时候,将优先的副本选举为主节点.

11. 分区重分配,写好json配置文件,生成重分配方案,进行保存执行.

    1. ~~~shell
       ## 根据reassign.json生成重分配的方案.
       ./kafka-reassign-partitions.sh  --zookeeper localhost:2181 --generate --topics-to-move-json-file reassign.json --broker-list 0,2
       ~~~

12. 使用kafka内置的sh来测试发送到主题的性能

    1. ~~~shell
       ## 生产能力测试
       /kafka-producer-perf-test.sh --topic testtopic --num-records 100 --record-size 100 --throughput -1 --print-metrics --producer-props bootstrap.servers=192.168.199.128:9092 acks=1
       ~~~

    2. 参数的解释

       1. ~~~shell
          100 records sent, 336.700337 records/sec (0.03 MB/sec), 51.83 ms avg latency, 291.00 ms max latency, 50 ms 50th, 51 ms 95th, 291 ms 99th, 291 ms 99.9th.
          ~~~

       2. records sent表示测试时发送的消息总数； records/sec表示以每秒发送的消息数来统计吞吐量，括号中的MB/sec表示以每秒发送的消息大小来统计吞吐量，注意这两者的维度； avg latency 表示消息处理的平均耗时； max latency表示消息处理的最大耗时； 50th 、 95th 、 99th和99.9th 分别表示50%、 95%、 99%和99.9%的消息处理耗时。

       3. ~~~shell
          ##消费能力测试
          ./kafka-consumer-perf-test.sh --topic testtopic --messages 100 --broker-list 192.168.199.128:9092
          
          ~~~

13. 并不是分区数越多,吞吐量就越好.一般可以设置为集群个数的倍数.

## 第五章 日志存储

1. Kafka使用了日志分段,一个log且分为多个logsegment.Log在物理上只是一个文件夹的形式存储,而每个logsegment对应一个日志文件和两个索引文件,以及可能的其它文件.
2. 日志关系
   - ![](.\picture\kafka\日志关系.png)
3. 日志数据结构发生了很多的改变,有很多的参数,例如:消息的键,消息的值,消息的长度.等到分析的时候在具体的看.
4. 为了快速的定位消息日志,使用了索引方式.kafka采用的稀疏索引的方式,没保存4k文件,才会创建一个索引,所以有些时候,不是所有的消息都会有索引.
5. 日志删除有三种方式:基于时间,基于日志大小,基于偏移量保留策略.

## 第六章 深入理解服务端

1. 时间轮:存储定时任务的环形队列,底层采用数组实现,数组中每一个元素可以存放一个定时任务列表.
2. 时间轮对于定时任务的添加和删除复杂度为O(1),性能高出DelayQueue.但是kafka在推进时间轮的时候,使用了DelayQueue,避免了推进任务时候的**空推进**.例如一个定时任务在200s之后,不用按照时间间隔一个一个的推进,而是使用DelayQueue直接进行推进.
3. kafka多个Broker会选举一个**控制器**,负责分区leader副本的选举工作.监控分区相关的变化,监听主题相关的变化,监听broker相关的变化.
4. 分区leader副本的选举:一般是从ISR集合中进行选举.

## 第七章 深入理解客户端

1. 分区分配策略:
   - RangeAssignor:将分区按照跨度进行平均分配,保证分区尽可能平均的分配给消费者.跨度是用消费者总数和分区总数进行整除运算获得.
   - RoundRobinAssignor:将消费组内所有消费者及消费订阅的所有主题的分区按照字典序排序,然后通过轮询的方式逐个将分区依次分配给每个消费者.如果每个消费订阅的消息相同,则这种方式可以实现平均.
   - StickyAssignor:(1)分区分配尽可能均匀.(2) 分区分配尽可能与上次分配的保持相同.
2. ConsumerCoordinator与GroupCoordinator之间最重要的职责是负责执行消费者在均衡的操作.
3. 在均衡原理:
   1. 第一阶段:FIND_COORDINATOR:消费者需要确认他所属的消费组对应的GroupCoordinator所在的broker,并创建于改Broker相互通信的网络连接.最终消费者保存了与消费组对应的GroupCoordinator节点信息.
   2. 第二阶段:JOIN_GROUP:消费者会向GroupCoordinator发送JoinGroupRequest请求,并处理响应.GroupCoordinator主要做两件事:
      1. 选举消费组的leader:如果没有第一个加入消费组的消费者即为消费组的leader,如果之前有,就从消费组随机选一个.
      2. 选举分区分配策略:少数服从多数为原则.
   3. 第三阶段:SYNC_GROUP:通过GroupCoordinator中间人同步分区分配策略给所有的消费者.
   4. 第四阶段:HEARTBEAT:消费者通过向GroupCoordinator发送心跳来维持他们与消费组的从属关系,以及他们对分区所有权的关系.
4. _consumer_offsets:集群中第一次有消费者消费消息时会自动创建主题__consumer_offsets.
   1. ![](.\picture\kafka\OffsetCommitRequest结构.png)
   2. retention_time:消息保存的时间:更具broker端配置的offsets.retention.minutes来确定,默认为7天.在2.0.0版本之前默认为1天.
5. 消息的传输保障:
   - 至多一次:消息可能会丢失,但绝不会重复传输.
   - 最少一次:消息绝不会丢失,但是会重复传输.默认为此等级.
   - 恰好一次:每条消息肯定会传输一次并仅传输一次.
6. 对于消息的重复传输以及丢失,和消息位移提交以及消费的处理顺序有关,如果处理在位移提交之前,如果发生故障后恢复,就会发生**重复消费**的问题,如果处理在位移提交之后,就会发生**消息丢失**的可能.
7. 幂等:通过properties.put("enable.idempotence",true)开启,但是需要确保客户端的retries,acks,max.in.flight.requests.per.connection这几个参数不被配置错.使用默认值就好,不需要刻意配置.
8. broker服务端维护为每一个生产者(<pid,分区>)维护一个版本号,通过比较版本号实现幂等的功能.pid为每一个新的生产者生成的.
9. 事务:
   1. 通过properties.put("transactional.id",transaction),应用程序必须提供唯一的transactionalId,并且开启幂等.
   2. 根据transactionalId,Kafka服务端也会生成一个PID,与之对应.对生产者而言,事务保证的语义比较强,对于消费者较弱.
10. 事务执行的步骤:
    1. **查找TransactionCoorinator**:TransactionCoorinator负责分配PID和管理事务.通过FindCoordinatorRequest请求实现,收到请求后会根据transactionalId查找对应的TransactionCoordinator节点.
    2. **获取PID:**幂等功能开启就会生成PID,而并非事务.通过InitProducerRequest请求发送给TransactionCoordinator,将PID保存在_transaction_state中.增加PID对应的producer_epoch(版本号).
    3. **开启事务**:kafkaProducer调用beginTransaction()方法可以开启一个事物.
    4. **Consume-Transform-Produce:**囊括了整个事务的数据处理过程.
       1. AddPartitionToTxRequest:生产者给一个新的分区发送数据之前,需要将此请求发送给TransactionCoordinator.主要的数据为<transactionId,TopicPartition>对应关系存储在_transaction_sate中.
       2. ProduceRequest:
       3. AddOffsetsToTXnRequest:KafkaProducer的SendOffsetsToTransaction()方法可以在一个事物批次里处理消息的消费和发送.这个方法发送AddOffsetsToTXnRequest:KafkaProducer,事务协调者(TransactionCoordinator)收到请求后,通过groupId来推导出在_consumer_offsets中的分区,之后会将这个分区保存在_transaction_state中.
       4. TxOffsetCommitRequest:这个也是SendOffsetsToTransaction方法执行的一部分.将本次消费位移传递给GroupCoordinator.
    5. **提交或者中止事务.**
       1. EndTxRequest
       2. WriteTxnMarkersRequest
       3. 写入最终的COMPLETE_COMMIT或者COMPLETE_ABORT.

## 第八章:可靠性探究

1. replica.lag.time.max.ms:默认值为10000,副本同步的最大时间,如果follower副本之后leader副本的时间超过此参数则判定为同步失败.
2. 副本会发生伸缩:本质是通过Zookeeper的监控副本的变化情况,同步到主节点进行变更.
3. 副本有本地副本和远程副本两种,本地副本存在本地的broker中,只有本地副本才有对应的日志(主节点).
4. Leader副本记录了所有follower的LEO,在接收到follow拉取日志的请求后,leader副本在发送消息之前会将follower副本的LEO更新.
5. **基于HW同步的消息丢失**:B为主副本,A为从副本,A,B都有m1,m2两条消息,HW都为1,当A从发送请求消息,并携带了自己的HW信息,B主根据请求更新了HW,在HW更新为2之前,B挂掉了,A成为主,B从进行同步,消息m2丢失.
6. **基于HW同步的消息不一致**:A主有m1,m2两条消息,且HW和LEO都为2,B从m1,且HW和LEO都为1.假设A,B同时挂掉,B先恢复成为主,B写入m3,并将LEO和HW更新为2,此时,A恢复,变为从,若根据HW同步,则A不用变更,但实质上是A中的m2,B主没有,B主的m3,A从也没有,出现了数据不一致的情况.
7. 基于HW副本同步,会出现消息丢失和消息不一致的场景.所以引入了Epoch.每当Leader变更一次,就会增加1,同时记录第一次写入消息的**偏移量,**增设一个矢量<LeaderEpoch=>StartOffset>相当于为Leader增设了一个版本号.以后根据偏移量进行同步消息.
   1. 基于epoch防止消息丢失:A发生重启后,A不先忙着截断日志而是先发送OffsetsForLeaderEpochRequest请求给B,B leader返回的当前的LEO,如果A和B中的LeaderEpoch不同,那么B会查找LeaderEpoch为LE_A+1对应的StartOffset并返回给A,也就是LE_A对应的LEO.即A收到2之后,发现与目前的LEO相同,也就不需要截断日志了,如果A成为Leader,则LE=0,变成LE=1,对应的消息m2此时得到了保留.
   2. 基于epoch防止数据不一致:A,B同时挂掉,B先恢复,此时LE0变成了LE1,A也恢复了,A为LE0,B根据LE0请求返回offset为1,m2删除,保持了数据的一致性.
8. ack参数
   1. ACK = 0 时 发送一次 不论leader是否接收
   2. ACK = 1 时，等待leader接收成功即可
   3. ACK = -1 时 ，需等待leader将消息同步给follower

## 第九章 kafka应用

1. 消费组一共有Dead,Empty,PreparingRebalance,completingRebalance,Stable这几种状态.组内没有消费者为空.

   1. ~~~shell
       ## 查看消费组状态.
       ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group flowgroup4 --state
      ~~~

   2. ~~~shell
      ## 查看消费组内消费者的信息以及分配情况
      ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group flowgroup4 --members --verbose
      ~~~

   3. ~~~shell
      ## 将消费唯一往前调整10,但是不执行. 只有--execute才是真正的执行
      ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group flowgroup4 --topic taw --reset-offsets --shift-by -10 --dry-run
      ~~~

   4. 除了--shift-by 还有--to-current 位移重置为当前 --from-file offsets.csv --execute 按照配置文件执行.

