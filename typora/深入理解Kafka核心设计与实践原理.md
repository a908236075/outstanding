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

2. 修改分区

   - ~~~shell
     #分区数修改为3
     ./bin/kafka-topics.sh --zookeeper localhost:2181/kafka --alter --topic demo --partitions 3  5
     ~~~

3. 分区数量与linux系统的文件描述符有关,一般上线是硬限制描述符4096个,可以手动设置增大.

4. 控制台模拟消费端

   - ~~~shell
     ## 从开始消费
     ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic flow_log --from-beginning
     ## 指定消费组
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic flow-event-log --consumer-property group.id=flowgroup2
     ~~~
     
   - ~~~shell
     ## 模拟生产者
   ./kafka-console-producer.sh --broker-list localhost:9092 --topic flow_log
     ~~~
   
   - 

### 注意:有时候kafka在server.properties的文件中配置连接的路径是localhost:2181而不是 localhost:2181/kafka 所以执行命令不能带/kafka.