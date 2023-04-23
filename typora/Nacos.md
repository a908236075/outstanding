# Nacos 配置、 服务注册及发现

## 配置

### 客户端

1. 注册监听器,监听器的注册在ClientWorker中处理，这块会创建一个CacheData对象.创建的CacheData缓存到ClientWorker中的一个Map中.
2. ClientWorker的构造函数里会去创建两个线程池，executor会每隔10ms进行一次配置变更的检查，executorService主要是用来处理长轮询请求的。
3. checkUpdateDataIds()该方法中，会将所有的dataId按定义格式拼接出一个字符串，构造一个长轮询请求，发给服务端，Long-Pulling-Timeout 超时时间默认30s，如果服务端没有配置变更，则会保持该请求直到超时，有配置变更则直接返回有变更的**dataId列表**。
4. 根据返回的dataId列表,用checkUpdateConfigStr()发起HTTP接口`/v1/cs/configs/listener`的调用。
5. checkListenerMd5()主要就是判断两个md5是不是相同，不同则调用safeNotifyListener()处理。
6. safeNotifyListener()方法主要就是调用监听器的receiveConfigInfo()方法，然后更新监听器包装器中的lastContent、lastCallMd5字段。

### 服务端

#### **客户端直接获取**

1.  **JDK的零拷贝**的方式 直接返回配置文件内容

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
2. dump()会将新的配置写入磁盘文件，更新md5，然后发布LocalDataChangeEvent事件。
3. LongPollingService会监听LocalDataChangeEvent事件，然后提交DataChangeTask。
4. DataChangeTask会找到监听这个配置的客户端，然后进行通知。

## 服务的注册

### 服务注册

1. 客户端启动时会将当前服务的信息包含ip、端口号、服务名、集群名等信息封装为一个Instance对象，然后创建一个定时任务，每隔一段时间向Nacos服务器发送PUT请求并携带相关信息。
2. nacos服务器端在接收到心跳请求后，会去检查当前服务列表中有没有该实例，如果没有的话将当前服务实例重新注册，注册完成后立即开启一个异步任务，更新客户端实例的最后心跳时间，如果当前实例是非健康状态则将其改为健康状态。
3. 心跳定时任务创建完成后，通过POST请求将当前服务实例信息注册进nacos服务器。
4. nacos服务器端在接收到注册实例请求后，会将请求携带的数据封装为一个Instance对象，然后为这个服务实例创建一个服务Service，一个Service下可能有多个服务实例，服务在Nacos保存到一个ConcurrentHashMap中`Map(namespace,Map(group::serviceName, Service))`

### 服务的发现

1. 服务发现大致过程：

    客户端定时从服务端拉取最新的服务列表数据，将服务列表加载到本地缓存。同样服务端也会定时向客户端推送服务列表数据。

### 服务的健康检查

1. 服务的健康检查分为两种模式：
   - 客户端上报模式：客户端通过心跳上报的方式告知nacos 注册中心健康状态（默认心跳间隔5s，nacos将超过超过15s未收到心跳的实例设置为不健康，超过30s将实例删除）
   - 服务端主动检测：nacos主动检查客户端的健康状态（默认时间间隔20s，健康检查失败后会设置为不健康，不会立即删除）