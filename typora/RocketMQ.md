# 环境准备

## 安装

1. 下载二进制的文件,不用自己编译.

   ~~~shell
   drwxr-xr-x. 2 root root   126 Sep 22 10:29 benchmark
   drwxr-xr-x. 4 root root  4096 Nov 24 22:01 bin
   drwxr-xr-x. 8 root root  4096 Sep 22 10:29 conf
   drwxr-xr-x. 2 root root  8192 Sep 22 10:29 lib
   -rw-r--r--. 1 root root 17327 Sep 21 19:58 LICENSE
   -rw-r--r--. 1 root root  1338 Sep 21 19:58 NOTICE
   -rw-r--r--. 1 root root 12265 Sep 21 19:58 README.md
   ~~~

   

## nameServer配置修改和启动

1. /bin目录下修改runserver.sh  -server -Xms512m -Xmx512m -Xmn256m

   ~~~shell
   choose_gc_options()
   {
       # Example of JAVA_MAJOR_VERSION value : '1', '9', '10', '11', ...
       # '1' means releases befor Java 9
       JAVA_MAJOR_VERSION=$("$JAVA" -version 2>&1 | awk -F '"' '/version/ {print $2}' | awk -F '.' '{print $1}')
       if [ -z "$JAVA_MAJOR_VERSION" ] || [ "$JAVA_MAJOR_VERSION" -lt "9" ] ; then
         JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
         JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8 -XX:-UseParNewGC"
         JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
         JAVA_OPT="${JAVA_OPT} -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m"
       else
         JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
         JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRefLRUPolicyMSPerMB=0"
         JAVA_OPT="${JAVA_OPT} -Xlog:gc*:file=${GC_LOG_DIR}/rmq_srv_gc_%p_%t.log:time,tags:filecount=5,filesize=30M"
       fi
   }
   ~~~

2. 启动

   ~~~shell
   nohup ./mqnamesrv & 
   [root@k8s-01 bin]# tail -f nohup.out 
   Java HotSpot(TM) Server VM warning: Using the DefNew young collector with the CMS collector is deprecated and will likely be removed in a future release
   Java HotSpot(TM) Server VM warning: UseCMSCompactAtFullCollection is deprecated and will likely be removed in a future release.
   The Name Server boot success. serializeType=JSON, address 0.0.0.0:9876
   ~~~

## Brocker配置修改和启动

1. 修改bin/runbroker.sh 配置

   ~~~shell
   JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m"
   JAVA_OPT="${JAVA_OPT} -XX:MaxDirectMemorySize=512m"
   ~~~

2. conf/broker.conf添加

   ~~~shell
   autoCreateTopicEnable=true
   namesrvAddr=localhost:9876
   ~~~

3. 启动

   ~~~shell
   ## 指定了配置文件
   nohup ./mqbroker -c ../conf/broker.conf &
   tail -f nohup.out 
   The broker[broker-a, 192.168.2.128:10911] boot success. serializeType=JSON and name server is localhost:9876
   ~~~

## 部署模型

![](.\picture\rocketmq\部署模型.png)

## 工作流程

![](.\picture\rocketmq\工作流程.png)

注意:消费者比生产者多了一步,向Broker 发送心跳,确保Broker存活的状态.

**同步**:Broker的master接收消息,等到消息同步到slave后,才会发从ack给producer.

**异步**:不用等消息同步到slaver就发送ack.

## 订阅模型

![](.\picture\rocketmq\架构图.png)

![](.\picture\rocketmq\订阅模型.png)

1. topic的一个消息队列,只允许消费组中的一个消费者进行消费.

![](.\picture\rocketmq\消息队列和消费组之间的关系.png)

2. topic的一个消息队列**可以**被不同的消费组进行消费.

![](.\picture\rocketmq\消息队列和消费组之间的关系2.png)



## 生产关系

​	![](.\picture\rocketmq\生产关系.png)

## 订阅关系

![订阅模型](.\picture\rocketmq\订阅关系.png)

## 模拟消息生产和消费

1. 生产

   ~~~shell
   export NAMESRV_ADDR=localhost:9876
   sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
   ## 消息体
   SendResult [sendStatus=SEND_OK, msgId=AC1100010801018FD1AC7B47697603A3, offsetMsgId=C0A8028000002A9F0000000000036C05, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=0], queueOffset=232]
   ~~~

2.消费

~~~shell
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
## 消息体
ConsumeMessageThread_please_rename_unique_group_name_4_11 Receive New Messages: [MessageExt [brokerName=broker-a, queueId=3, storeSize=241, queueOffset=51, sysFlag=0, bornTimestamp=1700836276784, bornHost=/192.168.2.128:50880, storeTimestamp=1700836276785, storeHost=/192.168.2.128:10911, msgId=C0A8028000002A9F000000000000C180, commitLogOffset=49536, bodyCRC=1797387810, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={CONSUME_START_TIME=1700836503310, MSG_REGION=DefaultRegion, UNIQ_KEY=AC1100010801018FD1AC7B47663000CE, CLUSTER=DefaultCluster, MIN_OFFSET=0, TAGS=TagA, WAIT=true, TRACE_ON=true, MAX_OFFSET=250}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 50, 48, 54], transactionId='null'}]] 

~~~

## 安装maven

1. 环境变量

   ~~~shell
   ## 配置文件内容
   export JAVA_HOME=/usr/local/java/jdk1.8.0_291
   
   export JRE_HOME=${JAVA_HOME}/jre
   
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
   
   export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
   
   export PATH=$PATH:${JAVA_PATH}:$MAVEN_HOME/bin
   
   export MAVEN_HOME=/ opt/maven/apache-maven-3.9.5
   ## 是配置生效命令
    source /etc/profile
   ~~~

2. 配置仓库地址和镜像

   ~~~xml
   <mirrors>
       <mirror>
             <id>alimaven</id>
             <name>aliyun maven</name>
             <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
             <mirrorOf>central</mirrorOf>        
       </mirror>
   </mirrors>
   ~~~


## Dashboard安装

~~~shell
## 拉镜像
docker pull apacherocketmq/rocketmq-dashboard:latest
## 运行
docker run -d --name rocketmq-dashboard -e "JAVA_OPTS=-Drocketmq.namesrv.addr=127.0.0.1:9876" -p 8080:8080 -t apacherocketmq/rocketmq-dashboard:latest
~~~

# 消费生产

**同步发送**:等待消息返回后在继续进行下面的操作.

**异步发送**:不等待消息返回值直接进入后续流程.broker将结果返回后调用callback函数,并使用CountDownLatch进行计数.

**单向发送**:只负责发送,不负责消息是否发送成功.

## 同步发送消息

~~~java
import com.sun.org.slf4j.internal.Logger;
import com.sun.org.slf4j.internal.LoggerFactory;
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.nio.charset.StandardCharsets;

public class ProducerExample {

    private static final Logger logger = LoggerFactory.getLogger(ProducerExample.class);

    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        String endpoint = "192.168.2.128:9876";
        String topic = "Simple";
        DefaultMQProducer producer = new DefaultMQProducer("SyncProducer");
        producer.setNamesrvAddr(endpoint);
        producer.start();
        for (int i = 0; i < 2; i++) {
            Message message = new Message(topic, "Tags", (i + "_syncProducer").getBytes(StandardCharsets.UTF_8));
            SendResult sendResult = producer.send(message);
            System.out.printf(i + "_消息发送成功%s%n", sendResult);
        }
        producer.shutdown();
    }


}
~~~

## 异步发送

~~~java
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.nio.charset.StandardCharsets;
import java.util.concurrent.CountDownLatch;

public class AsyncProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("AsyncProducer");
        producer.setNamesrvAddr("192.168.2.128:9876");
        producer.start();
        CountDownLatch countDownLatch = new CountDownLatch(100);
        for (int i = 0; i < 100; i++) {
            final int index = i;
            Message message = new Message("Simple", "ATag", (i + "AsyncProducer").getBytes(StandardCharsets.UTF_8));
            producer.send(message, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    countDownLatch.countDown();
                    System.out.println(index + "_消息发送成功_" + sendResult);
                }

                @Override
                public void onException(Throwable e) {
                    countDownLatch.countDown();
                    System.out.println(index + "_消息发送失败_" + e.getStackTrace());
                }
            });
        }
        countDownLatch.await();
        producer.shutdown();
    }

}

~~~



## 单向发送

~~~java
// 更改同步发送 SendResult sendResult = producer.send(message);   
//  变成 producer.sendOneway(message);
 producer.sendOneway(message);
~~~



## 顺序发送

~~~java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.nio.charset.StandardCharsets;
import java.util.List;

public class OrderProducer {
	// 按queue的顺序发送消息.
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer = new DefaultMQProducer("OrderGroup");
        producer.setNamesrvAddr("192.168.2.128:9876");
        producer.start();
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 10; j++) {
                Message message = new Message("Order", "TagA", ("order_" + i + "_step" + j).getBytes(StandardCharsets.UTF_8));
                SendResult sendResult = producer.send(message, new MessageQueueSelector() {
                    
                    @Override
                    public MessageQueue select(List<MessageQueue> list, Message msg, Object o) {
                        int id = (Integer) o;
                        int index = id % list.size();
                        return list.get(index);
                    }
                    // Object o 和 i 是同一个东西 表示orderId
                    // i 的解释 arg  Argument to work along with message queue selector.
                }, i);
                System.out.println("消息发送成功_" + sendResult);
            }
        }
        producer.shutdown();
    }
}

~~~



## 消息的消费(Push)

~~~java
package com.xiaobo.codingeveryday.rocketmq;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class ConsumerExample {

    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("SyncProducer");
        consumer.setNamesrvAddr("192.168.2.128:9876");
        consumer.subscribe("Simple", "*");
        consumer.setMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext context) {
                for (int i = 0; i < list.size(); i++) {
                    System.out.println(i + "_消息消费成功" + new String(list.get(i).getBody()));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.println("consumer stared %n");

    }
}
~~~

# 消息消费

## 拉模式一(不推荐)

~~~java
import org.apache.rocketmq.client.consumer.DefaultMQPullConsumer;
import org.apache.rocketmq.client.consumer.PullResult;
import org.apache.rocketmq.client.consumer.store.ReadOffsetType;
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.util.HashSet;
import java.util.Set;

public class PullConsumer {

    public static void main(String[] args) throws MQClientException {
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("PullGroup");
        consumer.setNamesrvAddr("192.168.2.128:9876");
        HashSet<String> topics = new HashSet<>();
        topics.add("Simple");
        topics.add("TopicTest");
        consumer.setRegisterTopics(topics);
        consumer.start();
        while (true) {
            consumer.getRegisterTopics().forEach(n -> {
                Set<MessageQueue> messageQueues = null;
                try {
                    messageQueues = consumer.fetchSubscribeMessageQueues(n);
                    messageQueues.forEach(l -> {
                        try {
                            long offset = consumer.getOffsetStore().readOffset(l, ReadOffsetType.READ_FROM_MEMORY);
                            if (offset < 0) {
                                offset = consumer.getOffsetStore().readOffset(l, ReadOffsetType.READ_FROM_STORE);
                            }
                            if (offset < 0) {
                                offset = consumer.maxOffset(l);
                            }
                            if (offset < 0) {
                                offset = 0;
                            }
                            PullResult pullResult = consumer.pull(l, "*", offset, 20);
//                            System.out.println("消息循环拉取成功_" + pullResult);
                            switch (pullResult.getPullStatus()) {
                                case FOUND:
                                    pullResult.getMsgFoundList().forEach(k -> {
                                        System.out.println("消息消费成功_" + k);
                                    });
                                    consumer.updateConsumeOffset(l, pullResult.getNextBeginOffset());
                            }
                        } catch (MQClientException e) {
                            e.printStackTrace();
                        } catch (RemotingException e) {
                            e.printStackTrace();
                        } catch (MQBrokerException e) {
                            e.printStackTrace();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    });
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
        }

    }
}

~~~

## 拉模式二

~~~java
import org.apache.rocketmq.client.consumer.DefaultLitePullConsumer;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

/**
 * 拉模式 随机获取一个queue消息
 */
public class LitePullConsumer {
    public static void main(String[] args) throws MQClientException {
        DefaultLitePullConsumer consumer = new DefaultLitePullConsumer("LitePullConsumer");
        consumer.setNamesrvAddr("192.168.2.128:9876");
        consumer.subscribe("Simple", "*");
        consumer.start();
        while (true) {
            List<MessageExt> messageExtList = consumer.poll();
            System.out.println("消息拉取成功");
            messageExtList.forEach(n->{
                System.out.println("消息消费成功_" + n);
            });
        }
    }
}
~~~

~~~java

import org.apache.rocketmq.client.consumer.DefaultLitePullConsumer;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.message.MessageQueue;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
/**
*	拉模式 指定一个queue消息
**/
public class LitePullConsumerAssign {
    public static void main(String[] args) throws MQClientException {
        DefaultLitePullConsumer consumer = new DefaultLitePullConsumer("LitePullConsumer");
        consumer.setNamesrvAddr("192.168.2.128:9876");
        consumer.start();
        Collection<MessageQueue> messageQueues = consumer.fetchMessageQueues("Simple");
        ArrayList<MessageQueue> messageQueueArrayList = new ArrayList<>(messageQueues);
        consumer.assign(messageQueueArrayList);
//        consumer.seek(messageQueueArrayList.get(0), 10);
        consumer.seek(messageQueueArrayList.get(1), 10);
        while (true) {
            List<MessageExt> messageExtList = consumer.poll();
            System.out.println("消息拉取成功");
            messageExtList.forEach(n->{
                System.out.println("消息消费成功_" + n);
            });
        }
    }
}

~~~

## 顺序消费

~~~java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.nio.charset.StandardCharsets;
import java.util.List;

public class OrderProducer {

    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer = new DefaultMQProducer("OrderGroup");
        producer.setNamesrvAddr("192.168.2.128:9876");
        producer.start();
        for (int i = 0; i < 5; i++) {
            for (int j = 0; j < 10; j++) {
                Message message = new Message("Order", "TagA", ("order_" + i + "_step" + j).getBytes(StandardCharsets.UTF_8));
                // object 和 i 要一致 id 对list.size() 取余
                SendResult sendResult = producer.send(message, new MessageQueueSelector() {
                    @Override
                    public MessageQueue select(List<MessageQueue> list, Message msg, Object o) {
                        int id = (Integer) o;
                        int index = id % list.size();
                        return list.get(index);
                    }
                }, i);
                System.out.println("消息发送成功_" + sendResult);

            }
        }
        producer.shutdown();

    }

}
~~~

~~~java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

public class OrderConsumer {

    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("SimpleConsumer");
        consumer.setNamesrvAddr("192.168.2.128:9876");
        consumer.subscribe("Order", "*");
        consumer.setMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext context) {
                for (int i = 0; i < list.size(); i++) {
                    System.out.println(i + "_消息发送成功_" + new String(list.get(i).getBody()));
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        consumer.start();
    }
}
~~~

1. 一般推荐推模式 

# 特殊的消息

## 延迟消息

~~~java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.nio.charset.StandardCharsets;
import java.time.LocalTime;

public class ScheduleProducer {

    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        String endpoint = "192.168.2.128:9876";
        String topic = "Schedule";
        DefaultMQProducer producer = new DefaultMQProducer("ScheduleProducer");
        producer.setNamesrvAddr(endpoint);
        producer.start();
        for (int i = 0; i < 10; i++) {
            Message message = new Message(topic, "Tags", (i + "_syncProducer").getBytes(StandardCharsets.UTF_8));
            // 主要是这里!!!
            // messageDelayLevel  1-18 对应 1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
            message.setDelayTimeLevel(2);
//            message.setDelayTimeMs(10000l);
            SendResult sendResult = producer.send(message);
            System.out.printf(i + "_消息发送成功%s%n"+ LocalTime.now(), sendResult);
        }
        producer.shutdown();
    }
}
~~~

~~~java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.message.MessageExt;

import java.time.LocalTime;
import java.util.List;

public class ScheduleConsumer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("ScheduleConsumer");
        consumer.setNamesrvAddr("192.168.2.128:9876");
        consumer.subscribe("Schedule", "*");
        consumer.setMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext context) {
                for (int i = 0; i < list.size(); i++) {
                    System.out.println(i + "_消息消费成功_" + LocalTime.now() + " " + new String(list.get(i).getBody()));
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        consumer.start();
        System.out.println("消费者启动成功!!");

    }
~~~

## 事务消息

## 批量数据的生产和消费

1. Consumer的MessageListenerConCurrently监听接口的consumeMessage()方法默认情况下只能消费一条消息,如果每次消费多条可以通过consumeMessageBatchMaxSize来指定,默认为1,每次拉取的消息数,可以通过Consumer的pullBatchSize属性来指定,默认为32,即每次拉取pullBatchSize条,拉取后分批次处理,每次处理consumeMessageBatchMaxSize条.存在消费失败的情况,需要合理设置并不是越大越好.

## 消息过滤

1. Tag过滤

   1. ~~~java
      consumer.subscribe("Schedule", "TagA || TagB");
      ~~~

2. Sql 过滤

   1. 支持过滤用户的属性信心,也就messag中的properties.

   2. .4.x默认不开启,需要设置enablePropertyFilter=true开启.

   3. ~~~java
      // 消息生产的时候提前设置了age属性 
      //  Message message = new Message("Simple", "ATag", (i + "AsyncProducer").getBytes(StandardCharsets.UTF_8));
      // message.putUserProperty("age",i+"");
      consumer.subscribe("Schedule",MessageSelector.bySql("age between 0 and 10"));
      ~~~

# 消息的存储和保存

1. Broker的store目录.

![store文件目录](.\picture\rocketmq\store文件目录.png)

---

# 2026 RocketMQ笔记

## 保证消息不丢失

### 生产阶段 (Producer)：确保消息成功到达 Broker

1. 在这个阶段，消息丢失的主要原因是网络抖动或 Broker 瞬间不可用。
   - **使用同步发送 (Sync Send)：** 坚决不能使用单向发送（Oneway）或完全不关心结果的异步发送。同步发送会阻塞等待 Broker 的 ACK 响应。只有拿到 `SendStatus.SEND_OK` 的状态码，才算真正发送成功。
   - **配置重试机制：** RocketMQ 默认在同步发送失败时会重试 2 次（`retryTimesWhenSendFailed = 2`）。结合之前提到的“故障规避机制”，它会自动切换到其他健康的 Broker 节点重试，极大提升发送成功率。
   - **分布式事务消息兜底：** 假设在你的**价格中心**系统中，更新了某项核心物料的基准价格（写入本地 MySQL），同时需要发消息通知下游的采购执行系统。为了防止“数据库写成功，但发消息失败”这种不一致情况，必须使用 RocketMQ 的**事务消息**（二阶段提交 + 补偿回查），确保本地事务和消息发送的强一致性。

### 存储阶段 (Broker)：确保消息落盘且有多副本

1. 当消息到达 Broker 的内存（PageCache）后，如果 Broker 机器突然断电，消息就会丢失。这里有两道防线：
   - **第一道防线：同步刷盘 (SYNC_FLUSH)**
     - *默认配置：* 异步刷盘（ASYNC_FLUSH）。消息写到操作系统的 PageCache 就向 Producer 返回成功，后台线程异步刷到磁盘。极快，但断电易丢数据。
     - *防丢失配置：* **同步刷盘**。消息必须真正调用 `fsync` 写入物理磁盘的 `CommitLog` 后，才向 Producer 返回 ACK。
     - *底层共鸣：* 这和 MySQL 的 `innodb_flush_log_at_trx_commit = 1`（每次事务提交都将 Redo Log 实时同步到磁盘）的设计哲学如出一辙，都是用 I/O 性能换取极致的可靠性。
   - **第二道防线：同步复制 (SYNC_MASTER) 或 Dledger 集群**
     - 即使落盘了，如果磁盘彻底损坏怎么办？需要多副本。
     - *传统主从架构：* 必须配置为 `SYNC_MASTER`（同步复制）。只有当 Master 和 Slave 都把消息写成功了，才给 Producer 返回成功。
     - *高可用架构：* 采用基于 Raft 协议的 **Dledger 模式**。消息写入时，必须满足“过半数节点”（Quorum机制，比如 3 节点中至少写入 2 节点）确认落盘，才算成功。这也是金融级消息中间件的标配。

### 消费阶段 (Consumer)：确保业务逻辑真正执行完

1. 消息安全到达了消费者，但这不等于消费成功。最常见的“丢消息”其实是开发人员用错了 API。
   - **手动提交 Offset，禁止异步“逃逸”：** RocketMQ 的 Consumer 默认是推模式（内部长轮询拉取）。千万不要在 `MessageListener` 里面开启一个异步线程去处理业务逻辑，然后主线程直接返回 `CONSUME_SUCCESS`。一旦异步线程抛异常或被 kill，这条消息的 Offset 已经被提交了，就再也找不回来了。
   - **正确处理异常与重试：** 只有当业务逻辑（例如下游系统根据价格变动更新了自身的计算规则）**完全彻底地执行成功，且本地数据库事务已经 commit**，才能返回 `CONSUME_SUCCESS`。 如果遇到数据库死锁、网络超时等问题，应该抛出异常或返回 `RECONSUME_LATER`，让 RocketMQ 的重试机制（默认 16 次，采用阶梯退避时长）介入。
   - **死信队列 (DLQ) 最终兜底：** 如果一条消息重试了 16 次依然失败，说明系统出现了代码 Bug 或严重的数据异常。此时 RocketMQ 会把消息送入死信队列（`%DLQ%ConsumerGroupName`）。你需要通过人工介入、监控告警和后台管理工具对死信消息进行排查和手动重发，确保业务闭环。

要实现 **绝对的不丢失**，你需要一套王炸组合： `同步发送` + `事务消息` + `SYNC_FLUSH (同步刷盘)` + `SYNC_MASTER (或 Dledger 集群)` + `严格等待本地事务成功的消费端 ACK`。

## 消息的存储机制（PageCache 与 mmap）

- 

## 保证消息按照顺序消费

RocketMQ 默认并不保证全局有序，因为那意味着整个 Topic 只能有一个 Queue，这会彻底扼杀分布式系统的并发吞吐量。它真正落地并在工业界广泛采用的是**分区顺序（局部顺序）**。

为了保证这三个事件的顺序，RocketMQ 在底层通过“**三步走 + 三重锁**”的机制来实现：

### 第一步：生产阶段（消息路由到同一个 Queue）

要保证顺序，前提是同一组强相关的消息必须进入物理上的同一个队列（MessageQueue）。

- **自定义路由规则：** 生产者在发送消息时，需要实现 `MessageQueueSelector` 接口。以物料价格变动为例，我们可以提取 `skuId` 或 `协议ID` 作为 Sharding Key。通过对 Key 的 Hash 值与该 Topic 的 Queue 总数取模，确保同一个 SKU 的变更事件一定会落入同一个 Queue 中。
- **同步发送：** 顺序消息的发送**必须**使用同步发送（`Sync`）。如果是异步发送，网络延迟可能导致先发的请求后到达 Broker，从源头就乱序了。

### 第二步：存储阶段（Broker 天然按序存储）

RocketMQ 的底层存储设计对于单个 Queue 内的消息是天然保证顺序的。 Broker 收到同一个 Queue 的消息后，会按到达的先后顺序追加写入到 `CommitLog` 中，并按顺序构建 `ConsumeQueue` 索引。因此，只要生产者没乱，存储端就不会乱。

### 第三步：消费阶段（核心难点：多重锁机制）

这是保证顺序消费最复杂、也是面试最核心的环节。消费者必须使用 `MessageListenerOrderly` 而不是 `MessageListenerConcurrently`。

RocketMQ 底层通过**三把锁**来保证消费的绝对单线程执行：

1. **Broker 端的分布式锁：** 当消费者启动并进行负载均衡（Rebalance）分配到某个 Queue 后，它必须先向 Broker 发起请求，给这个 Queue 加锁。**只有拿到了 Broker 侧的锁，消费者才能开始从该 Queue 拉取消息。** 这保证了同一个消费者组内，不可能有两个实例同时拉取同一个 Queue 的消息。
2. **消费者端的 ProcessQueue 锁：** 拉取到消息后，消息会被放入消费者本地内存的 `ProcessQueue` 中。RocketMQ 为每个 `ProcessQueue` 分配了一把 `ConsumeLock`。多线程拉取时，必须先获得这把锁，才能将消息交给业务逻辑处理。
3. **消费者端的 线程同步锁（对象锁）：** 在最终调用 `MessageListenerOrderly` 执行具体业务代码前，还会对消费这批消息的上下文（Context）加 `synchronized` 锁，确保同一个 Queue 的消息在本地是单线程逐条被消费的。

### 💡 资深工程师必考：顺序消息的“坑”与避坑指南

当你向面试官阐述完上述机制后，如果能主动抛出以下生产环境中的痛点并给出方案，会极大提升你的技术深度得分：

**痛点 1：消费阻塞（Queue 拥堵）** 普通并发消息如果消费失败，会被投入重试队列（`%RETRY%`），不会影响后续消息的处理。 但在**顺序消费模式**下，如果价格中心抛出的某条“修改协议价”消息因为业务异常（如数据库死锁）消费失败，返回了 `SUSPEND_CURRENT_QUEUE_A_MOMENT`，RocketMQ **绝对不能**跳过它去消费下一条。它会在本地不断重试（默认无限次，或者最大重试次数），导致该 Queue 后续的所有消息全部被堵死。

- **架构级解决方案：** * **业务侧降级：** 设定最大重试次数（如 3-5 次）。如果依然失败，必须在代码中捕获异常，将这条“毒药消息”连同其上下文记录到死信数据库表或 Redis 中，然后强制返回 `SUCCESS` 让队列继续往前走。后续通过定时任务或人工介入核对这条物料的数据状态。
  - **监控告警：** 必须对 ConsumeQueue 的滞后程度（Lag）配置严格的告警机制。

**痛点 2：Broker 宕机与锁过期切换** 如果持有 Broker 锁的消费端实例突然 Crash 了，锁没有释放怎么办？

- Broker 端对 Queue 的锁有过期时间（默认 60 秒）。发生 Crash 后，最长经过一分钟，其他消费者实例就能重新抢到该 Queue 的锁接管消费，但这期间该 Queue 的消费是暂停的。

**痛点 3：热点问题（数据倾斜）** 如果采购云中某个核心供应商的大宗物料（同一个 SKU）变动极其频繁，由于它们都被 Hash 到同一个 Queue，会导致对应的那台 Broker 和 Consumer 实例负载远高于集群内其他机器。

- **解决方案：** 这需要在业务层做折中，如果并发量极高，可以考虑在 Hash Key 中加入时间片窗口（比如 `skuId + yyyymmddHH`），牺牲跨时间窗口的绝对顺序，换取散列度，但这需要业务逻辑自身具备较强的状态容错能力

## 保证消息的事物

RocketMQ 采用了 **“两阶段提交（2PC） + 补偿回查机制”** 来保证事务的绝对可靠。

它的完整流转分为以下核心步骤：

### 发送 Half Message（半消息）

价格中心（Producer）首先向 RocketMQ Broker 发送一条“价格变更”的 Half Message。

- **什么是半消息？** 这条消息已经成功写入了 Broker 的磁盘，但是对下游的消费者（订单中心）是**不可见**的。它被临时存在了一个内部的 Topic（`RMQ_SYS_TRANS_HALF_TOPIC`）里。

### 执行本地事务

Broker 返回“半消息发送成功”的 ACK 后，价格中心的系统开始执行本地 MySQL 的事务（执行 `UPDATE` 语句并 `COMMIT`）。

###  提交或回滚 (Commit / Rollback)

根据本地 MySQL 事务的执行结果，价格中心向 Broker 发送二次确认：

- **如果本地事务成功：** 发送 `Commit` 指令。Broker 会把那条半消息从内部 Topic 移到真正的目标 Topic 中，此时订单中心就能消费到这条消息了。
- **如果本地事务失败（如抛出异常）：** 发送 `Rollback` 指令。Broker 会直接丢弃（逻辑删除）这条半消息，就当什么都没发生过，下游也不会收到任何通知。数据保持一致。

------

**核心兜底机制：Broker 状态回查 (Check)**

上面的 3 步看起来很完美，但如果**第 3 步由于网络断开，或者价格中心的 JVM 突然 Crash，导致 Broker 迟迟没有收到 Commit 或 Rollback 怎么办？**

这就轮到 RocketMQ 最精妙的**回查机制**出场了：

1. **主动轮询：** Broker 内部有一个定时任务，会定期扫描那些一直处于“半消息”状态的孤儿消息。
2. **发起回查：** 当发现某条半消息悬而未决时，Broker 会主动通过网络反向请求价格中心（Producer），询问：“你之前发的那条价格变更消息，对应的本地数据库事务到底有没有执行成功？”
3. **业务自证：** 价格中心需要实现一个回查接口（`TransactionListener.checkLocalTransaction`）。在这个接口里，通常会去查询 MySQL 中特定的“事务流水表”或者直接查询业务数据状态。
   - 如果查到价格已经更新：返回 `Commit`。
   - 如果查不到更新记录：返回 `Rollback`。
   - 如果还不确定（可能还在执行中）：返回 `Unknown`，Broker 下次还会再来问。

通过这个回查兜底，即使在极端宕机场景下，只要本地事务成功了，消息最终一定会被发出去；只要本地回滚了，消息最终一定会被取消。

## 延迟消息

## NameServe无状态 互相不通信 这么保证注册信息的实时性

1. 它们的数据同步完全依赖于 **Broker 的主动汇报**。
2. **同步机制（全量注册）：** Broker 启动时，会轮询配置好的所有 NameServer 节点，和**每一个** NameServer 建立长连接。随后，Broker 每隔 **30 秒** 会向所有的 NameServer 发送一次心跳包，心跳包里包含了该 Broker 的所有路由信息（比如有哪些 Topic、多少个 Queue 等）。
3. NameServer 可能返回“脏数据”（把已经挂掉的 Broker 返回给了 Producer）。
   - 当 Producer 按照轮询策略选中了一个 Queue，并向对应的 Broker 发送消息。如果发现发送失败（连接超时等），Producer 会触发重试逻辑（默认重试 2 次）。
   - 在重试时，RocketMQ 有一个非常聪明的**故障规避机制（FaultItem）**：它会把刚刚发送失败的那个 Broker 暂时“拉黑”一段时间，在接下来的重试或后续发送中，**优先选择其他健康的 Broker 上的 Queue**，从而避免消息发送被卡死。
4. NameServer 是一个极简的“地址簿”，靠 Broker 主动汇报来维持（最终）同步；Producer 随机找一个 NameServer 复印一份地址簿带在身上（本地缓存），然后自己根据轮询或自定义算法挑一个 Queue，最后直接把消息拍在对应 Broker 的门脸上。

















