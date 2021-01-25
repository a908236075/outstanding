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

