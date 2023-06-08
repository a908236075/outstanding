# Nacos 配置、 服务注册及发现

## 配置

### 客户端

1. **注册监听器**,监听器的注册在**ClientWorker**中处理，这块会创建一个**CacheData**对象(它包含了服务的基本信息，比如服务名、IP地址、端口号、健康状态、权重、集群等信息).创建的CacheData缓存到ClientWorker中的一个ConcurrentHashMap中,其中的key是通过服务名和集群名共同组成的.
2. ClientWorker的构造函数里会去创建两个线程池，executor会每隔10ms进行一次配置变更的检查，executorService主要是用来处理**长轮询**请求的。
3. checkUpdateDataIds()该方法中，会将所有的dataId按定义格式拼接出一个字符串，构造一个长轮询请求，发给服务端，Long-Pulling-Timeout 超时时间默认30s，如果服务端没有配置变更，则会保持该请求直到超时，有配置变更则直接返回有变更的**dataId列表**。
4. 根据返回的dataId列表,用checkUpdateConfigStr()发起HTTP接口`/v1/cs/configs/listener`的调用。
5. checkListenerMd5()主要就是判断两个**md5**是不是相同，不同则调用safeNotifyListener()处理。
6. safeNotifyListener()方法主要就是调用监听器的receiveConfigInfo()方法，然后更新监听器包装器中的**lastContent**、**lastCallMd5**字段。

### 服务端

#### **客户端长轮询的方式**:

1. ClientLongPolling()会启动一个延时30执行的任务，如果30s内配置没有变更(其实是等待DataChangeTask)，**任务就会执行，对客户端进行响应**，如果30s内配置发生了变更，此任务就会被取消。客户端的请求会放在allSubs的队列中.

~~~java
public void run() {
	// 延时30s执行
	asyncTimeoutFuture = ConfigExecutor.scheduleLongPolling(new Runnable() {
		@Override
		public void run() {
			try {
				getRetainIps().put(ClientLongPolling.this.ip, System.currentTimeMillis());

				// Delete subsciber's relations.
				allSubs.remove(ClientLongPolling.this);

				if (isFixedPolling()) {
					... ...
				} else {
					LogUtil.CLIENT_LOG
						.info("{}|{}|{}|{}|{}|{}", (System.currentTimeMillis() - createTime), "timeout",
							  RequestUtil.getRemoteIp((HttpServletRequest) asyncContext.getRequest()),
							  "polling", clientMd5Map.size(), probeRequestSize);
					// 超时直接返回
					sendResponse(null);
				}
			} catch (Throwable t) {
				LogUtil.DEFAULT_LOG.error("long polling error:" + t.getMessage(), t.getCause());
			}

		}

	}, timeoutTime, TimeUnit.MILLISECONDS);

	// 将客户端端缓存至队列中
	allSubs.add(this);
}

~~~

2. generateResponse()会将变更配置的dataId和group新信息返回给客户端，**并不会返回具体的配置内容**，内容会由客户端来查询。

#### 配置变更通知客户端

1. publishConfig()会将配置保存到数据库中，并发布ConfigDataChangeEvent事件。通知所有节点配置变更通知.
2. AsyncNotifyService监听了ConfigDataChangeEvent事件，然后提交了AsyncTask任务来对Nacos集群中的节点进行通知配置的变化.
2. AsyncTask.run()会调用Nacos集群中的所有节点（包含自己）的Http接口/v1/cs/communication/dataChange来通知配置的变化。
2. notifyConfigInfo()主要负责将变化的配置从数据库中查询出来，然后更新本地的文件。
2. dump()会将新的配置写入磁盘文件，更新md5，然后发布LocalDataChangeEvent事件。
3. LongPollingService会监听LocalDataChangeEvent事件，然后提交DataChangeTask。
4. DataChangeTask会找到监听这个配置的客户端，然后进行通知。

#### **客户端直接获取**

1.  **JDK的零拷贝**的方式 直接返回配置文件内容

## 服务的注册

### 服务注册

#### 客户端

1. 客户端启动时会将当前服务的信息包含ip、端口号、服务名、集群名等信息封装为一个Instance对象，调用NacosServiceRegistry注册方法进行注册

   1. ~~~java
      public void register(Registration registration) {
      
      	if (StringUtils.isEmpty(registration.getServiceId())) {
      		log.warn("No service to register for nacos client...");
      		return;
      	}
      
      	// 获取NamingService
      	NamingService namingService = namingService();
      	String serviceId = registration.getServiceId();
      	String group = nacosDiscoveryProperties.getGroup();
      
      	// 将Registration封装为Instance
      	Instance instance = getNacosInstanceFromRegistration(registration);
      
      	try {
      		// 注册实例
      		namingService.registerInstance(serviceId, group, instance);
      		log.info("nacos registry, {} {} {}:{} register finished", group, serviceId,
      				 instance.getIp(), instance.getPort());
      	}
      	catch (Exception e) {
      		if (nacosDiscoveryProperties.isFailFast()) {
      			log.error("nacos registry, {} register failed...{},", serviceId,
      					  registration.toString(), e);
      			rethrowRuntimeException(e);
      		}
      		else {
      			log.warn("Failfast is false. {} register failed...{},", serviceId,
      					 registration.toString(), e);
      		}
      	}
      }
      ~~~

2. NacosNamingService 第一次创建时会通过groupname + servicename创建一个心跳实例并启动心跳检测定时任务。

   1. ~~~java
      public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {
      	NamingUtils.checkInstanceIsLegal(instance);
      	// groupname 和 servicename 组合,DEFAULT_GROUP@@serviceName
          // 默认groupname为DEFAULT_GROUP
      	String groupedServiceName = NamingUtils.getGroupedName(serviceName, groupName);
      	if (instance.isEphemeral()) {
      		// 构建心跳实例
      		BeatInfo beatInfo = this.beatReactor.buildBeatInfo(groupedServiceName, instance);
      		// 启动心跳定时任务
      		this.beatReactor.addBeatInfo(groupedServiceName, beatInfo);
      	}
      
      	// 服务注册
      	this.serverProxy.registerService(groupedServiceName, groupName, instance);
      }
      ~~~


#### 服务端

1. 接收到客户端的请求后,有ServiceManager进行管理,如果没有这个Service 就创建一个,最后将实例instance封装在Service里面.

   1. ~~~java
      public void createEmptyService(String namespaceId, String serviceName, boolean local) throws NacosException {
      	createServiceIfAbsent(namespaceId, serviceName, local, null);
      }
      
      public void createServiceIfAbsent(String namespaceId, String serviceName, boolean local, Cluster cluster)
      	throws NacosException {
      	Service service = getService(namespaceId, serviceName);
      	if (service == null) {
      
      		Loggers.SRV_LOG.info("creating empty service {}:{}", namespaceId, serviceName);
      		service = new Service();
      		service.setName(serviceName);
      		service.setNamespaceId(namespaceId);
      		service.setGroupName(NamingUtils.getGroupName(serviceName));
      		// now validate the service. if failed, exception will be thrown
      		service.setLastModifiedMillis(System.currentTimeMillis());
      		service.recalculateChecksum();
      		if (cluster != null) {
      			cluster.setService(service);
      			// cluster在这里创建
      			service.getClusterMap().put(cluster.getName(), cluster);
      		}
      		service.validate();
      
      		// 将service放入serviceMap，并初始化
      		putServiceAndInit(service);
      		if (!local) {
      			addOrReplaceService(service);
      		}
      	}
      }
      ~~~

2. 创建新的实例要与现有的服务实例进行合并.ServiceManager#addInstance.key是通过namespace,serviceName生成的.相同的key的instance比较他们的ip,如果相同就进行替换.将实例列表数据放入缓存中，然后添加一个实例变更的任务。

   1. ~~~java
      public void addInstance(String namespaceId, String serviceName, boolean ephemeral, Instance... ips)
      	throws NacosException {
      
      	String key = KeyBuilder.buildInstanceListKey(namespaceId, serviceName, ephemeral);
      
      	// 又查了一次Service？？？
      	Service service = getService(namespaceId, serviceName);
      
      	synchronized (service) {
      		// 将缓存中的实例和注册中的实例合并得到instanceMap，最后将当前实例加入instanceMap
      		List<Instance> instanceList = addIpAddresses(service, ephemeral, ips);
      
      		Instances instances = new Instances();
      		instances.setInstanceList(instanceList);
      
      		// 将实例列表放入缓存中
      		/**
                   * @see DelegateConsistencyServiceImpl#put(String, Record)
                   */
      		consistencyService.put(key, instances);
      	}
      }
      
      ~~~

   2. ~~~java
      private List<Instance> addIpAddresses(Service service, boolean ephemeral, Instance... ips) throws NacosException {
      	return updateIpAddresses(service, UtilsAndCommons.UPDATE_INSTANCE_ACTION_ADD, ephemeral, ips);
      }
      
      public List<Instance> updateIpAddresses(Service service, String action, boolean ephemeral, Instance... ips)
                  throws NacosException {
      
      	// 从缓存中查询实例
      	Datum datum = consistencyService
      		.get(KeyBuilder.buildInstanceListKey(service.getNamespaceId(), service.getName(), ephemeral));
      
      	// 从注册表中查询实例
      	List<Instance> currentIPs = service.allIPs(ephemeral);
      	// <ip, instance>
      	Map<String, Instance> currentInstances = new HashMap<>(currentIPs.size());
      	// 实例ID集合
      	Set<String> currentInstanceIds = Sets.newHashSet();
      
      	for (Instance instance : currentIPs) {
      		currentInstances.put(instance.toIpAddr(), instance);
      		currentInstanceIds.add(instance.getInstanceId());
      	}
      
      	// 合并缓存中的实例和注册表中的实例
      	Map<String, Instance> instanceMap;
      	if (datum != null && null != datum.value) {
      		instanceMap = setValid(((Instances) datum.value).getInstanceList(), currentInstances);
      	} else {
      		instanceMap = new HashMap<>(ips.length);
      	}
      
      	for (Instance instance : ips) {
      		if (!service.getClusterMap().containsKey(instance.getClusterName())) {
      			Cluster cluster = new Cluster(instance.getClusterName(), service);
      			cluster.init();
      			service.getClusterMap().put(instance.getClusterName(), cluster);
      			Loggers.SRV_LOG
      				.warn("cluster: {} not found, ip: {}, will create new cluster with default configuration.",
      					  instance.getClusterName(), instance.toJson());
      		}
      
      		if (UtilsAndCommons.UPDATE_INSTANCE_ACTION_REMOVE.equals(action)) {
      			// 从instanceMap中删除实例
      			instanceMap.remove(instance.getDatumKey());
      		} else {
      			// 将实例放入到instanceMap中
      			Instance oldInstance = instanceMap.get(instance.getDatumKey());
      			if (oldInstance != null) {
      				instance.setInstanceId(oldInstance.getInstanceId());
      			} else {
      				instance.setInstanceId(instance.generateInstanceId(currentInstanceIds));
      			}
      			instanceMap.put(instance.getDatumKey(), instance);
      		}
      
      	}
      
      	if (instanceMap.size() <= 0 && UtilsAndCommons.UPDATE_INSTANCE_ACTION_ADD.equals(action)) {
      		throw new IllegalArgumentException(
      			"ip list can not be empty, service: " + service.getName() + ", ip list: " + JacksonUtils
      			.toJson(instanceMap.values()));
      	}
      
      	// 将合并后的实例返回
      	return new ArrayList<>(instanceMap.values());
      }
      ~~~

3. 实例变更的任务通过异步的Notifier#run方法进行更新处理的.核心的任务是将实例instance封装在Service里面更新服务.将缓存中获取的实例列表按clusterName进行分组，最后以cluster为维度进行更新注册表。

4. 服务端怎么保证注册表的高并发读和写？

   1. 注册表的写是异步的，而且是通过一个任务来执行，这样保证只有一个线程进行写，不会有并发。
   2. 写注册表时是使用新的对象直接覆盖原来的引用，类似CopyOnWrite机制，不影响读。
   3. CopyOnWrite其核心概念就是：  数据读取时直接读取，不需要锁，数据写入时，需要锁，且对副本进行操作。那么当数据的操作以读取为主时，我们便可以省去大量的读锁带来的消耗。同时为了能让多线程操作List时，一个线程的修改能被另一个线程立马发现，CopyOnWriteList采用了Volatile关键词来进行修饰，复制副本进行数据更改,更改后将副本传值给Volatile修饰的变量.

### 服务的发现

#### 客户端 :

1. 所谓服务发现就是指客户端从[注册中心](https://so.csdn.net/so/search?q=注册中心&spm=1001.2101.3001.7020)获取记录在注册中心中的服务信息.
2. NacosReactive**Discovery**Client进行处理,通过openFent调用NacosServiceDiscovery#getInstances,
3. NacosNamingService会对实例列表进行简单的选择(NacosNamingService#selectInstances),**实际开发过程中实例的选择是由Ribbon或者LoadBalancer实现。**Nacos服务端会返回所有的实例列表，包含不健康状态的实例，具体要不要选择不健康的实例由客户端决定。

4. 先从本地缓存中查询服务对应的实例列表(HostReactor#getServiceInfo)，如果本地缓存中没有就会实时的去查询Nacos服务端，最后会开启一个定时任务定时去Nacos服务端查询。
5. 为了获取实时的服务列表，Nacos客户端不仅有定时任务1秒执行一次去获取服务列表，还开启了一个UDP端口，用于接收Nacos服务端服务实例信息变更的推送。
6. 总结:启动的时候通过httpClientProxy类调用api,运行的时候通过定时任务每秒进行获取,还有一个UDP接收推送.

#### 服务端

1. Nacos客户端会通过调用接口/nacos/v1/ns/instance/list来查询服务端对应服务的实例列表。
1. 查询服务的实例列表直接查询的是注册表，不同namespace，不同group之间的service是无法调用的，同一个service下的不同Cluster可以调用。

### 服务的健康检查

#### 服务端

1. 启动的时候开启了心跳监测任务ClientBeatCheckTask, 遍历服务中的所有实例，如果[心跳](https://so.csdn.net/so/search?q=心跳&spm=1001.2101.3001.7020)时间超过15s就将健康状态标记为false，如果心跳时间超过30s就会删除实例。
2. 将健康状态标记为false时会发布InstanceHeartbeatTimeoutEvent和ServiceChangeEvent两个事件。InstanceHeartbeatTimeoutEvent没有地方监听此事件。ServiceChangeEvent由本身发布事件的PushService监听.
3. 健康状态标记为false,会将此服务在注册列表中删除.
4. PushService监听了ServiceChangeEvent事件。当服务变更时，Nacos服务端会通过**UDP**通知所有监听此服务的客户端。这里为了使客户端能够实时的知道服务的实例状态变更了，又为了不增加服务器的压力，所以使用了UDP，因为UDP不需要建立连接，直接发送一个报文即可，不管客户端有没有收到。即使客户端没有收到，客户端也有一个定时任务每隔5s来查询服务的实例列表。

## 服务消费

### 基本概念

1. 根据负载均衡发生位置的不同,一般分为**服务端负载均衡和客户端负载均衡**。 服务端负载均衡指的是发生在服务提供者一方,比如常见的nginx负载均衡 而客户端负载均衡指的是发生在服务请求的一方，也就是在发送请求之前已经选好了由哪个实例处理请 求。我们在微服务调用关系中一般会选择**客户端**负载均衡.
2. 客户端是引入了discover的后台,与之相对应的是Nacos的服务端.

### 服务订阅

1. ![](..\picture\微服务\Nacos_服务订阅.png)
2. 通过请求流程图我们发现, 在创建serviceId的上下文时，加载了类RibbonNacosAutoConfiguration 该类又会触发 NacosRibbonClientConfiguration类的加载
3. NacosRibbonClientConfiguration 会自动创建NacosServerList**,NacosServer可以理解为客户端从注册中心获取的服务信息NacosService.**
4. 负载均衡是怎么更新服务列表的:负载均衡器ZoneAwareLoadBalancer 在初始化时调用了父类DynamicServerListLoadBalancer的 serverListUpdater.start(updateAction)方法,该方法30秒执行一次NacosServerList.getUpdatedListOfServer方法.

## 重要组件

1. NacosNamingService
   1. ![](..\picture\微服务\Nacos_NacosNamingService.png)

## 实战问题

1. Nacos的架构是怎样的？它有哪些重要组件？

   - Nacos是一个分布式的、高可用的服务发现和配置管理系统，它的架构主要由以下四个组件构成：

     1. Naming Service（命名服务）：负责服务的注册、发现和注销等功能。
     2. Configuration Service（配置服务）：提供动态配置管理功能，支持各种数据格式的配置信息。
     3. Cluster Manager（集群管理）：负责集群节点的管理和调度，实现高可用、负载均衡和故障转移等功能。
     4. Console（控制台）：提供Web界面，用于对Nacos的各项功能进行管理和监控。

     其中，Naming Service和Configuration Service是Nacos的核心组件，主要用于服务的注册和配置管理。Cluster Manager用于实现高可用和负载均衡，保证服务的稳定性和可靠性。而Console则提供了对Nacos的可视化管理界面，便于用户进行服务管理和配置管理等操作。

     在Nacos的架构中，Naming Service和Configuration Service都是基于Raft协议实现的分布式一致性算法，使用Raft协议可以保证系统的高可用性和数据的一致性。而Cluster Manager则使用基于选举的Leader/Follower模式实现，通过选举Leader节点来保证系统的高可用性和负载均衡。

2. Nacos如何实现配置服务？

   1. 客户端

      - 注册监听器,在**ClientWorker**中实现监听过程,创建一个**cacheData**对象,这个对象包含服务名,服务名、IP地址、端口号、健康状态权重集群等信息,保存在ConcurrentHashMap,其中的key是通过服务名和集群名共同组成的.
      - **ClientWorker**会创建线程池,线程池会每10ms调用一次**长轮询**(有配置更新直接返回,没有配置更新的时候到间隔时间后返回)的配置更新检查任务,长轮询的默认返回时间是30s,10ms的设置保证了更新的及时性.
      - 如果有更新返回dataId,客户端通过此参数调用nacos的config/listen 接口,获取配合文件内容.
      - checkListenerMd5()主要就是判断两个**md5**是不是相同，不同更新**lastContent**、**lastCallMd5**字段。
      - **为什么长轮询任务不会积压:**
        1. 不会导致大量长轮询任务积压的原因是,长轮询任务并非每次都是30s才返回,如果有更新会立即返回.但是不是主要的原因.
        2. 如果配置没有更新的情况下,单看一个任务的执行还是30s返回,其实压力还是在服务端,服务端采用到了队列加缓存的方式,当检测到有配置更新的时候直接返回给客户端,当遇到变更时,会应答当前时间之前未超时的所有的任务,等于说一个返回,就会将数成千上万个请求返回.
        3. 服务端采取了异步执行,也是创建了29.5s的周期任务,同时采用了缓存+队列+线程池等技术,保证了节点的性能.

   2. 服务端

      1. 处理长轮询请求:
         - 启动一个定时任务29.5s,如果配置没有变更(其实是等待DataChangeTask)，**任务就会执行，对客户端进行响应空值**，如果30s内配置发生了变更，此任务就会被取消。客户端的请求会放在**allSubs**的队列中.
         - generateResponse()会将变更配置的**dataId和group**新信息返回给客户端，**并不会返回具体的配置内容**，内容会由客户端来查询。
         - 当配置变更会触发publishConfig方法,触发ConfigDataChangeEvent监听事件,通过异步任务进行配置文件更新操作.
         - AsyncTask.run() 会通知所有节点(包括它自己)配置文件改变.
         - notifyConfigInfo()主要负责将变化的配置从数据库中查询出来，然后更新本地的文件。
         - dump()会将新的配置写入磁盘文件，更新md5，然后发布LocalDataChangeEvent事件。
         - 最后触发DataChangeTask会找到监听这个配置的客户端，然后进行通知。
      2. 客户端直接获取
         - **JDK的零拷贝**的方式 直接返回配置文件内容

3. 服务的注册

   1. AbstractAutoServiceRegistration实现了ApplicationListener接口,容器启动后开始注册
   2. 客户端将ip,端口号,服务名,分组名进行封装,使用**NacosServiceRegistry**注册.会通过接口调用的方式进行注册.而且NacosNamingService通过通过groupname + servicename构建一个心跳任务**BeatReactor**(每5s发送一次),这个是注册时候的心跳将会将整个beatInfo(ip,端口号,分组,权重等)对象发送给服务端.
   3. 服务端通过ServiceManager进行管理,是将instance转变为一个Service并封装在Map中,key是通过namespace,serviceName生成的,比较ip,如果相同进行替换.并添加一个实例变更的任务,实例变更的任务通过异步的**Notifier**#run方法进行更新处理的.核心的任务是将实例instance封装在Service里面更新服务.将缓存中获取的实例列表按clusterName进行分组，最后以cluster为维度进行更新注册表。更新的是实例的状态,这个用到的是CopyOnWrite机制.
   4. 服务端怎么保证注册表的高并发读和写？
   
      1. 注册表的写是异步的，而且是通过一个任务来执行，这样保证只有一个线程进行写，不会有并发。
      2. 写注册表时是使用新的对象直接覆盖原来的引用，类似CopyOnWrite机制，不影响读。
      3. CopyOnWrite其核心概念就是：  数据读取时直接读取，不需要锁，数据写入时，需要锁，且对副本进行操作。那么当数据的操作以读取为主时，我们便可以省去大量的读锁带来的消耗。同时为了能让多线程操作List时，一个线程的修改能被另一个线程立马发现，CopyOnWriteList采用了Volatile关键词来进行修饰，复制副本进行数据更改,更改后将副本传值给Volatile修饰的变量.
   
4. 服务发现

   1. 客户端
      1. 所谓服务发现就是指客户端从[注册中心](https://so.csdn.net/so/search?q=注册中心&spm=1001.2101.3001.7020)获取记录在注册中心中的服务信息.
      2. **NacosReactiveDiscoveryClient**进行处理,通过openFlient调用NacosServiceDiscovery#getInstances,
      3. NacosNamingService会对实例列表进行简单的选择(NacosNamingService#selectInstances),**实际开发过程中实例的选择是由Ribbon或者LoadBalancer实现。**Nacos服务端会返回所有的实例列表，包含不健康状态的实例，具体要不要选择不健康的实例由客户端决定。

      4. 先从本地缓存中查询服务对应的实例列表(HostReactor#getServiceInfo)，如果本地缓存中没有就会实时的去查询Nacos服务端，最后会开启一个定时任务定时去Nacos服务端查询。
      5. 为了获取实时的服务列表，Nacos客户端不仅有定时任务每秒执行一次去获取服务列表，还开启了一个UDP端口，用于接收Nacos服务端服务实例信息变更的推送。
      6. 总结:启动的时候通过httpClientProxy类调用api,运行的时候通过定时任务每秒进行获取,还有一个UDP接收推送.
   2. 服务端
      1. Nacos客户端会通过调用接口/nacos/v1/ns/instance/list来查询服务端对应服务的实例列表。
      2. 查询服务的实例列表直接查询的是注册表，不同namespace，不同group之间的service是无法调用的，同一个service下的不同Cluster可以调用。



































