# Spring cloud 微服务

## 各个部件版本

### 搭建环境

#### 	热部署

​	

### 服务注册中心 Nacos Eureka(×) (/juˈriːkə/ )

##### cap理论:

​	C:Cnsistency:强一致性.

​	A:Availability: 可用性

​	P:Partition tolerance:分区容错性.

任何软件都只能占用其中两个,是不可能同事满足三个的,三角理论.

Eureka:AP    Zookeeper/consul占用的CP

### 服务调用 Ribbon(/ˈrɪbən)   LoadBalancer

#### Ribbon

​	1.负载均衡:

​			集中式:zookeeper   在服务端使用负载均衡

​			进程内:Ribbon  在本地实现负载均衡.

​	2.自定义算法的时候,不能放在MapperScan扫描的包下.需要在启动类同级的报下定义自定义算法.

​	3. 负载均衡算法:rest接口第几次请求数%服务器集群总数量=实际调用服务器位置下标,每次服务重启后rest接口计数从1开始.

### 服务调用2 OpenFeign ( /feɪn/ )

#### 	作用:声明式的web客户服务端.

​	1.完成对服务提供方的接口绑定,简化了使用spring cloud ribbon,自动封装服务调用客户端的开发量.

#### 服务降级 resilience4j (/rɪˈzɪl,iəns) sentienl  Hystrix(×)

Jmeter 模拟线程请求的数量.

### 服务网关 gateway  Zuul(×)

gateway:非阻塞,异步的编程.

​	路由 断言  过滤

### 服务配置 Nacos Config(×)

bootstrap.yml是系统级的,比application.yml优先级更高.不会被本地配置覆盖.

Stream:屏蔽底层细节,实现与不同的消息中间件交互.



### 服务总线 Nacos Bus(×)

Nacos: Naminig configuration service 的缩写.一个更易于构建云原生应用的动态服务发现,配置管理和服务管理平台.

单机模式启动 Nacos 命令: startup.cmd -m standalone

配置文件的名称:

```
${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
最终配置成的文件名称:nacos-config-client-dev.yaml
```

定义文件名称的时候,需要加上后缀名.yaml  **注意不是yml**

#### Sentinel

fallback:是处理运行时异常的方法.

blackHander:负责配置异常的结果.

如果两个异常我们都进行了配置,如果同时出发,最后生效的是blackHandler.

### Seata

1. 解决分布式事务的问题.
2. 组成
   1. 全局唯一的事物ID.
   2. Transaction Coordinator (TC):事务协调器,维护全局事务的运行状态,负责协调并驱动全局事务的提交和回滚.
   3. Transaction Manager(TM):控制全局事务的边界,负责开启一个全局事务,并最终发起全局提交和全局回滚的决议.
   4. Resource Manager(RM):控制分支事务,负责分支注册,状态汇报,并接收事务的协调指令,驱动分支事务的提交和回滚.