# 性能监控

## Prometheus

1. 架构图
   ![](.\picture\性能监控\Prometheus架构图.png)
2. 使用了多个exporters进行监控,例如Mysql Serve exporter, Memcached exporter专家.
3.  Java服务集成Prometheus
   1. 引入sentry-client  SDK包.
   2. 第一步进行注册,将java服务注册到Prometheus中,这样Prometheus才会定时的去拉取数据.
   3. 通过切面通知,将服务的参数例如内存使用量进行读取,返回给Prometheus,存储在TSDB中.

## Grafana

1. 是性能监控的前台页面,通过PromQL类似于Sql的语句,到Prometheus中进行查询展示.