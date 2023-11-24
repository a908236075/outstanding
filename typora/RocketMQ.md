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

   

   

   





