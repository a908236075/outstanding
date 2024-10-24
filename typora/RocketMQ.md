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

# 4.x 消费生产



















