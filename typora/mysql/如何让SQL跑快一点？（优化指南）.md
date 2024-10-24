![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52YMFOzJicj5q8MutTLDStcEf6Vhtb3PjMBnmCyWMXmEMQ0icrVEGiaoiaRw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

阿里妹导读

这篇文章主要探讨了如何在阿里云MaxCompute（原ODPS）平台上对SQL任务进行优化，特别是针对大数据处理和分析场景下的性能优化。

一、前言：SQL从提交到运行

SQL代码提交到ODPS上后，会经过一段时间的运行，得到最终的运行结果。SQL优化，就是对这个运行过程进行优化，主要表现在：**缩短运行时间、减小运行消耗成本**。在正式进行任务优化之前，可以先了解ODPS上SQL从提交到运行的整个链路。

先看看我们熟悉的ODPS，作为阿里巴巴自主研发的大数据处理平台，ODPS提供海量数据的计算处理和分析服务，使用户不用了解数据计算存储细节，而可以直接进行数据查询和处理。下图简单介绍了ODPS的组成及ODPS上提交作业和运行作业流程。

**MaxCompute（ODPS）四大组成部分：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKnoPpBqDLz3LROkDkxuHxs5ibYc6RQC746AQy25dwmCrQypia6WNckOsy9n4W8XjpQ8cgC246ic2jZA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**ODPS上提交作业和运行作业流程图：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52r2p5dJNMAiakEYStSqGrKjmL7t8vmlibL7EMUiblITknDS6HATLYUQZEw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

二、优化第一步，先看logview

**Logview查看：**

进行SQL任务优化，读懂logview是关键，可以在D2开发平台下面的面板点击链接键查看logview。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52aHx3eg3ImyhDibVpdMSiczT6CAut8pTS0PQBhfCdH3Ck6Fib2c0WxuORA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Logview展示了Fuxi Job Dag图、Fuxi Task和Fuxi Instance等信息。他们的关系如下：一个SQL任务可以由多个Fuxi Job构成，一个Fuxi Job可以由多个Fuxi Task构成，一个Fuxi Task可以由多个Fuxi Instance构成。在logview面板的下方可以查看每个Fuxi Instance的明细情况。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52y2gPSwIwrVqLG49TEqUHZtPDKVnicAjSDsP2ySVOBpdutzyOKqepUlQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ODPS中的Fuxi Task一般是有四种字母：M\\R\\J\\C开头，分别代表四种任务类型。Fuxi Job Dag图中的Fuxi Task以这四种字母开头，后面连接的序号一般代表该Task和其依赖的直接上游Task编号。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ525ONXYJfN9ibaDc4p9Nl70ybeYMInjBoIErDgWutQTB3jY2yLsRg1w1A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在Fuxi Job Dag图中，可以直观看到各Fuxi Task的Instance个数，如下图所示左边代表运行中的（0），中间表示已经运行完成的（86），右边表示该Task总实例数（86）。R/W分别代表读取Read和写入Write。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52FLmANxhunNAsx8xT6xCM8t7hOpBNwNKDiaSCexzDktZibEgvibrbXx42Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Logview定位：**

定位问题的一般步骤：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52akssSYc9TbdQUEsHyiamMeMftJ2cUpwRxd0KFQyaPZlWLHDuicBUL6Iw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

找到运行时长较长的Fuxi Instance后，可以通过：

1、查看Input output判断是否有数据膨胀。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKnoPpBqDLz3LROkDkxuHxshticI1uVkoToGJcB3jKAhsc74pHn5ezVR2lrVWSSbHMporwuGLEnicBA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2、通过Long-Tails、Data-Skews看是否有长尾、数据倾斜。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ528vZdl6HwwibY8tnU1ZOnMCURgiaRSaxUaY08CqjJsX424LDibMcsEdLWA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、根据StdOut编码定位具体的sql片段：StdOut 可以看到实例中的各算子， DAG 图中双击运行时间最长的 Fuxi Task，展示出算子图，点击 StdOut 中对应的具体算子，即可定位具体的sql片段。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52UkMQRQBJedbKJVA0Rq4qwnhCfz7Giaa4MFts8Sibuia87pkH5gu28zuibQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Logview效果查询：**

左边面板Latency看运行时长，上方Summary面板中通过resource cost查耗费，换算公式：耗费= resource cost cpu / 1440 \* 1.52元。

**Logview 实用小技巧补充：**

### 1、五彩的热力图

在Fuxi Job Dag图中，看板默认展示是Progress Chart，可以点击切换成Task Time Heat Chart，**根据热力图找到运行时间最长的 Task（橙色），此时，Dag 图中线的粗细也能直观表示数据输入输出量大小。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52t3XSDPI13G9ka6ia8icYibnfuYEW7kQJp4XFpvVkj5ribPCTcXbRpHSiaqQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2、神奇的Latency Chart

Latency Chart 直观展示了Task中各实例运行情况，可以这样理解：蓝线的宽度代表各实例的运行时长，越宽说明运行时间越长，图中的毛刺一般就是最长实例之一。**高度越高代表该实例越晚开始运行，斜率越高表示资源压力越大。**可以结合Job Dag中Task方框的开始结束时间和这里的Latency Chart图综合判断是否是资源获取耗时太久。截图中的Latency Chart图斜率较高，说明对应时间段资源获取压力较大。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52hkjBLfvgvwXkDHXgAibwbfhFlRojTdU8iatCOxFIjPpkhYeoHyEC1Cpw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52cT8bGJ91cib7fuxB2gCzLUR6xsvrdVd4715ukSIicjdibsicia97icTJQOsg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 3、具体的算子（Operator）

Fuxi Task 在逻辑上可以被拆分成多个算子。**双击主面板 DAG 图中的某个 Task，可以看到算子详情，其中橙色代表耗时占比。**（如果是 C 开头命名的 Task 就没有算子信息，正如上文提到的，该类 Task 表示动态调度控制，不涉及计算）

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52eicG5gEZJG8oIyh9y1fWONPwfUq4t5hDKTJ16UDwbV0XMTLibJa9eJicQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

常见的算子有：

<table><colgroup><col width="183"><col width="554"></colgroup><tbody><tr><td colspan="1" rowspan="1"><p><span>算子名</span></p></td><td colspan="1" rowspan="1"><p><span>简介</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>TableScan</span></p></td><td colspan="1" rowspan="1"><p><span>读取物理表（表名、分区信息等）</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>TableSink</span></p></td><td colspan="1" rowspan="1"><p><span>将数据写入物理表</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>Calc</span></p></td><td colspan="1" rowspan="1"><p><span>根据&nbsp;where&nbsp;条件过滤，并查询需要的基础字段</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>StreamLineWrite</span></p></td><td colspan="1" rowspan="1"><p><span>按照分区、排序字段将数据写入，数据落盘过程（shuffle&nbsp;的前半部分）</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>StreamlineRead</span></p></td><td colspan="1" rowspan="1"><p><span>读取数据（shuffle&nbsp;的后半部分）</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>Project</span></p></td><td colspan="1" rowspan="1"><p><span>查询需要的基础字段（做字段拆分等简单加工）</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>Expand</span></p></td><td colspan="1" rowspan="1"><p><span>根据&nbsp;&nbsp;grouping sets&nbsp;&nbsp;中给定的维度交叉逻辑，对记录进行复制</span></p><p><span>其中，$groupingid 字段会作为标签来区分不同的记录副本</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>HashAgg</span></p></td><td colspan="1" rowspan="1"><p><span>哈希聚合，基于哈希表</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>SortedAgg</span></p></td><td colspan="1" rowspan="1"><p><span>排序聚合，基于全局排序</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>MergeAgg</span></p></td><td colspan="1" rowspan="1"><p><span>合并聚合，通常是在局部聚合结果上的合并处理</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>Filter</span></p></td><td colspan="1" rowspan="1"><p><span>指定条件过滤，通常对应于 where 条件</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>MergeJoin</span></p></td><td colspan="1" rowspan="1"><p><span>一般表示常规&nbsp;join&nbsp;过程</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>HashJoin</span></p></td><td colspan="1" rowspan="1"><p><span>一般表示&nbsp;mapjoin&nbsp;过程</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>ConditionalMapjoin</span></p></td><td colspan="1" rowspan="1"><p><span>实现 mapjoin</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>AdhocSink</span></p></td><td colspan="1" rowspan="1"><p><span>（临时查询）结果输出</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>Window</span></p></td><td colspan="1" rowspan="1"><p><span>产出窗口函数的结果（rank()、row_number()等）</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>ReduceSink</span></p></td><td colspan="1" rowspan="1"><p><span>Task 间数据分发操作（当前 Task 的结果传递给另一个 Task），explain 结果会显示分发的 key、value 以及输出结果的排序方式等</span></p></td></tr><tr><td colspan="1" rowspan="1"><p><span>Limit</span></p></td><td colspan="1" rowspan="1"><p><span>query 语句中‘limit’语句块的逻辑</span></p></td></tr></tbody></table>

其中，TableScan 和StreamLineWrite 一般耗时会较长。

### 4、信息量大的 Summary

在主面板中切换到Summary，可以看到各 Task 的执行情况。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52Yp33WK8vJvWjdhcEiaJUllpwbj6SS2NxNCYAKd8EJvoVFLlzicQlOw5g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

由于Task中的Instance是并发进行的，所以Task的执行时间并不等于各实例运行时间之和，一般会比最长实例运行时间长。Task的输入输出数据量一般会等于其各实例的输入输出数据量之和。

**举个小栗子实战：**

看完上述 Logview 小技巧后，我们举个栗子快速定位 SQL 运行慢的原因：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52WH3icLsavhXcY58ic1mhVm5u4QdhbKaN64djKibDLVmzS8QbFibHH59wgA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

看 DAG 图知道是J4\_1\_2\_3 和R5\_4 运行时间长。分别双击点开看算子详情：

J4\_1\_2\_3 部分，发现是在 Expand 和 HashAgg 之后的StreamLineWrite4 数据写入环节耗时较长（看橙色占比）：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52LBScWHoJO2ll9QkBcTkTJcBwlsGAFqcVPhr3bRCrzqFVMG30yH8siaw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

R5\_4 部分，StreamLineRead4 耗时较长：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52qibo8ydrvGf5Mqib9k5Jrh95BFBNzrYk5aD3fELgib2icbrlTAlB4zIWJw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

说明可能是代码中存在的 GROUPING SETS 中维度交叉组合过多，Expand 阶段数据规模膨胀，Hash聚合count(distinct)消耗一定时间，且输出较多数据量。导致J4\_1\_2\_3 中的写入和R5\_4 的读取都耗时较长，可以考虑减少一些没必要的维度交叉组合。

三、SQL运行慢的常见原因

**1、资源紧张，任务优先级比较低**

点击logview看Fuxi Job Tag图（如下），如果每个方框（也就是Fuxi Task）都是白色，说明实例还没运行，这个时候可以看面板左边的Queue Length，显示计算资源排队队列个数。同时点进去Waiting Queue，可以查看当前所有作业快照，通过不同维度（项目、配额等）聚合来看资源情况：一般绿色代表资源情况较好，黄色次之，红色更次。发现资源紧张后，也可以联系相应运维同学协助解决。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52HiaJiauMpzNxdSeIbgQVGKGkq72rN3I7cYjEiaMZZRYhOd97wlria4QRRw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Priority代表优先级，和运维中心实例DAG图中的优先级数字是9-n的关系，截图这里数字越大优先级越低。可以通过参数设置（set odps.instance.priority=0;）来提升开发环境任务的优先级。

**2、参数设置不合理**

ODPS系统常用的参数设置：

通过map split块大小设置来控制Map Instance个数(set odps.sql.mapper.split.size=256;)，一般会结合：

(set odps.sql.mapper.merge.limit.size=64;)一起使用，后者可以使每个Map Instance读入文件更均匀。

通过(set odps.sql.joiner.instances=-1; set odps.sql.reducer.instances=-1;)分别设置Join Task Instance和Reduce Task Instance数量，当每个task运行时间较长且没有发生长尾时可以考虑增加实例数。

通过：

(set odps.sql.groupby.skewindata=true/false;)开启Group by优化，在数据倾斜情况下可以使用。

参数设置不合理也会导致任务运行缓慢。比如当Instance并行度设置过大，会使得资源等待时间较长，并行度设置过小，任务执行时间又可能过长。实际操作中可以通过二分法在开发环境中测试Instance的合适大小。

**3、SQL语句不当**

SQL关键语句执行顺序：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52cFllqMm2D0j1b5gMTmZqcEqE0DNCwYQKfJiavb6lPVEz4mIp9AzPm3g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中from、select在解析过程进行，where分区字段在解析阶段直接裁剪，非分区字段在map阶段过滤。因此在写SQL语句时，避免写select \*，充分利用where限制分区，添加条件限制数据量等。当表数据量很大时，可以考虑先将where条件写进子查询内，再和其他表进行join。

**4、数据倾斜**

数据倾斜是SQL优化中常见的情况，是指在MapReduce模型中大量value值集中在少部分reducer中处理的情况，由于少部分reducer处理的数据量过大，从而延长了整个任务执行的时间。出现数据倾斜的原因一般有以下几种：

1、join中关联出现热点key（相关reducer耗时较长）；

2、join关联字段中空值过多（处理空值的reducer耗时超过平均值）；

3、group by中出现热点key，某些键值数据量过多（比如在shop\_id+order\_id表中计算各商户的订单量，热门商户就可能是热点key）；

4、join中count distinct中特殊值过多；

可以用odps中的mapjoin hint语法、手动切分热点、设置skewjoin参数(set odps.sql.skewjoin=true;)等来对数据倾斜进行处理。

**5、数据计算量大**

当数据量非常大时，ODPS需要更多的计算资源来并行处理数据。如果没有适当调整资源分配（如增加实例数、使用更高级别的实例类型），任务可能会因为资源不足而运行缓慢甚至失败。此时可以看下是否能一开始就对数据进行一些裁剪。比较经典的场景是根据业务理解添加适当的where条件、提前group by去重等来限制数据量，比如流量表限制log来源等，可以有效减少数据读取量。

ads应用表中，当cube的维度很多时，各个维度的枚举值组合起来，进行count(distinct)计算时耗费会比较高。此时可以先从业务上进行优化，比如裁剪一些用不到的维度组合。如果裁剪之后还是跑的很慢，且groupby与distinct字段值都均匀时，可以考虑用group by代替count distinct。

四、SQL任务优化实战

**1、大表join小表：用Mapjoin Hint处理**

在odps中，可以通过显式指定Mapjoin Hint提示，添加（/\*+ MAPJOIN(小表1,小表2,小表3) \*/）语句来提升SQL运行效率。开启Mapjoin后，odps会在map阶段将小表的数据加载在内存中，在处理大表的每一行记录时，直接使用本地的小表数据进行join操作，大大提高了join的效率。

举个栗子：

通过如下SQL计算城代超会数：

```
select ds,count(ditsinct user_id) as 城代超会数from(SELECT  /*+ MAPJOIN(c) */                    b.ds                    ,b.user_id            FROM   table_b b            INNER JOIN  table_c c            ON      b.district_id = c.district_id)group by ds;
```

其中b表是一张每天分区有几亿数据的大表，c表是维表每天数据几百万，在b表和c表join的时候，通过增加/\*+ MAPJOIN(c) \*/语句，可以显著提升运行效率。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52rjrUznUzW0vpcvQY4PwN7GqHHvNgR3JIQqtUx8lfUE6aNguxf8AW9Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在用了mapjoin hint后，在join之前先对小表进行了reduce处理，随后的join节省了几个小时。然而，**MapJoin也有其适用条件和限制**，比如小表的大小需要适中，不能太大，否则会占用过多的内存资源，影响集群性能。可以通过参数设置(set odps.sql.mapjoin.memory.max=2048;)来增加小表储存内存。

MapJoin优化适用于左连接、右连接、内连接，不可用于全连接，应用在左连接时，大表必须为左表；应用在右连接时，大表必须为右表。

**2、用双重group by代替count(distinct)**

当数据量大且count distinct对象key的分布比较均匀时，可以用双重group by代替count distinct。

举个栗子：

```
select ds,count(distinct item_id) as item_cntfrom item_tablegroup by ds;
```

改写后：

```
select ds,count(item_id) as item_cntfrom(select ds,item_idfrom item_tablegroup by ds,item_id)group by ds;
```

在ads应用层常见的cube表中，当数据量较大且去重计算对象键值分布较均匀时，可将要去重计算的对象如user\_id等放入grouping\_sets中，先去重再count，以提升运行效率。

代码举例：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52icRD2maITibkSme2ypYX56dDSU282haonmSocZyn14B0FAv4icHtCxDaQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**3、进阶版本，处理多重count distinct：**

当代码中count distnct对象有多个时，可以将聚合对象分开，再将最后数据union all或者join起来，但当聚合对象过多时，join过程可能会耗时较长。此时可以考虑用TRANS\_COLS函数，一行变多行进行计算。

函数的语法使用如下：

```
select TRANS_COLS(参数1,列1,列2,...,聚合对象1,聚合对象2) AS (idx,列1,列2,...,新聚合对象)
```

其中参数1表示相关的维度数。

举个栗子，假设有3个维度：

city\_id,client\_code,is\_new\_user，计算对象分别是user\_id和imeisi，此时语句可以写成：

```
select TRANS_COLS(3,city_id,client_code,is_new_user,user_id,imeisi) AS (idx,city_id,client_code,is_new_user,visitor_id);
```

在该SQL语句作用下，原来的3行数据就变成了6行，其中vistior\_id代表新的聚合对象。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ523Mak6pvx4q2NVSINL8Ywl2eI8MmOoattAVpwTLvxlKl4XhviaJ22CyA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

优化实战：

某SQL中核心代码：count distinct较多且条件复杂，且有两个聚合对象：user\_id、imeisi：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52iasdrnjyGNHm9W4vI8aUhomRIpMQia8O8sx9DdnWuaGgPjAzwkh5mpcA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用TRANS\_COLS函数后的代码：

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ5258qRvVLncZRB5NoMD1Gu1udfUDT12DIKh1ZB3ebUjvPcD3vviaYmhKw/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

写入TRANS\_COLS函数后，聚合对象由多个变成1个，此时再用2中双重group by代替count distinct的方法，SQL跑起来耗时和耗费均减少。

在该例子中，写入TRANS\_COLS函数后的运行成本减少将近一半。

**4、临时表较大，进行拆分**

当表里有复杂临时表耗时较长时，可以根据业务特性或者需要将临时表进行拆分，将大段资源消耗拆分成多段，避免因为资源等待而耗时。

**5、合理采用UDF函数**

用户自定义函数（UDF，User Defined Function）可以帮助我们实现特定的数据处理能力，有一些适当的UDF函数可以显著帮助我们节省运行时长。比如函数：rb\_build\_or\_agg，针对用户去重计算相关指标优化效果显著。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL8wlxJDXTUeC1vtzQHCJ52e4Da48Zibou7aoVW5bjakfNYNGaXicH0FPe7QymrSyOuoYic2srYqLZ6Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

但当UDF函数涉及大量循环或者没有充分利用其并行处理能力时，可能会使SQL任务整体运行时长增加或者资源消耗增大。SQL任务中有bi\_udf:bi\_get\_date、bi\_date\_format等相关函数时，可以替换成内置函数。详情参考BI\_UDF更换治理优化实战相关文章。

**6、Hash分桶优化**

在ODPS中，当涉及大数据集的join操作时，使用Hash分桶（Bucket）技术也可以优化数据处理性能。Hash分桶可以通过将数据预处理分配到不同的“桶”中，使得具有相同或相似连接键的数据尽可能地落在同一个桶里，从而减少后续连接操作时的数据扫描范围，提升处理效率。

具体语句是：

```
alter table table_xxx clustered by(column_name) sorted by(column_name) into 2108 buckets;
```

这里buckets的数量并不是越多越好，而是要根据查询性能和数据量大小权衡选择，可以在开发环境上观测运行效果来试验最合适的分桶数。

五、结语

在大数据处理和分析场景下，数据治理不可或缺。本文阐述了业务实战中常见慢SQL场景和优化方式，欢迎大家交流，最后感谢公司和团队里各位大佬的指导！