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
6. 

