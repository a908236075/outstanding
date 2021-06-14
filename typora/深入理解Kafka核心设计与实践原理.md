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

4. 消费者参数:

   - AR集合:分区中所有副本的统称为AR.   AR=ISR+OSR.
   - ISR集合:所有与leader保持一定同步的副本组成.
   - OSR集合:与leader副本同步滞后过多的副本组成OSR(out-of-Replica)

5. Key会被kafka server用来做为将数据分区的参数.

6. 消费完数据也需要提交commit,来更新队列里面处理消息的位置(下一次拉取的消息的位置).一般都是手动提交.

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
     ./bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic topic-demo --unavailable-partitions
     ~~~
   
2. 创建主题 test 3个副本 2个分区

      ~~~shell
   ./kafka-topics.sh --zookeeper localhost:2181 --create --topic test --replication-factor 3 --partitions 2
   ~~~

3. 修改分区

   - ~~~shell
     #分区数修改为3
     ./bin/kafka-topics.sh --zookeeper localhost:2181/kafka --alter --topic demo --partitions 3  5
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
   
      1. ![](D:\develop\gitHub\outstanding\typora\picture\kafka\生产者客户端的整体架构.png)
      2. 由主线程和Sender线程组成,主线程会将消息保存到**消息累加器中**,累加器的大小可以通过buffer.memory来配置,不同的分区中底层的数据结构是多个双端队列.通过设置batch.size参数设定producerBatch的大小.
      3. Sender从RecordAccumulator获取缓存消息后,会进一步将<分区,Deque<ProducerBatch>>的保存形式转换为<Node,List<ProduderBatch>>的形式.
      4. inFlightRequest通过未回应的请求判断Node节点中负载最小的那一个.
      5. 记录集群的节点,分区信息的都称为元数据.

## 第三章 消费者

1. 消息投递的模式:**点对点模式和发布订阅模式**.当所有的消费者都隶属于同一个消费组,消息均衡的投递给每一个消费者,这时使用点对点模式.
2. subscribe()和assign()都能订阅消息,但subscribe具有消费者自动再均衡的功能.
3. 消费模式有推模式和拉模式两种.
