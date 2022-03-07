# Flink尚硅谷视频学习

## 基本概念

1. 有界数据和无界数据:无界数据是指离线的数据.
2. 三个层级:第一层封装好直接用的api,第二层为一般处理离线的数据.第三层是拿到的数据最为全面.![](.\picture\flink\flink分层.png)

3. Flink VS Spark
   1. Flink是毫秒级别,Spark是秒级别.
   2. Flink数据模型是数据流,而spark是RDD模型.