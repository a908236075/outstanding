![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJLbJg2zzuIkM3SXJnzyHibq1U8d1zJA1R3GOFtpx6LwdoqUdXYS2ONjtS6uzn7awhJtsQApfI2qSg/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

阿里妹导读

分享作者在使用Arthas火焰图工具进行Java应用性能分析和优化的经验。

看到标题是不是很多人在想是不是标题党了，是也不是👻~，请听我细细道来~

我们的应用代码是用Java写的，因此使用的火焰图工具是Arthas，下面的分析也是基于此。

Arthas火焰图使用

官方文档：

https://arthas.aliyun.com/doc/profiler.html

**启动火焰图分析**

```
$ profiler startStarted [cpu] profiling
```

**停止**

```
$ profiler stop --format flamegraphprofiler output file: /tmp/test/arthas-output/20211207-111550.htmlOK
```

如上所示默认会生成一个html的火焰图文件，指定输出格式相关可参考官方文档。

**火焰图示例**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2njMgx7AI5eOebyl82MicT3pibmWgZibBF0doBDD8xsK9V3bAPTyQNVUlxA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

火焰图横轴代表CPU的占用时间，横轴越宽代表CPU占用越多，鼠标移动上去也可以看到这个方法究竟占用了多少CPU。

纵轴代表调用栈，火焰越高代表调用栈越深。

其中绿色部分代表Java代码，黄色部分代表JVM C++代码，橙色部分代表内核态C语言代码，红色代表用户态C语言代码。

如何分析火焰图(附实战)

事情是这样的，我们的业务简单来说就是监听发货消息后执行一系列操作，分自动和批量两种方式，批量的大用户进行业务操作时，会同时有几万单、十几万单的产生，相当于大促时的流量了，因此cpu占用总是有尖刺，有时单机甚至能到80+%。而且日常流量时感觉cpu占用和流量数据相比也有些偏高，因此决定使用火焰图分析下。

**从上往下看**

下面采用两个在优化过程中比较典型的两个案例。

#### **案例一**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2nIzLmfUhmT1Nsic3ISGlvtAsBCnsq8Iw32vIypKd7p0FfNibleqaRW7aQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

火焰图分析最简单的方式就是找大平顶，如果一个方法占用比较耗时、调用次数很多，那么他的横轴一定是比较宽的，体现到火焰图上就是一个大平顶。

图中红框是业务代码的执行，其中蓝框是初步定位到的耗时操作，点开后可以看到左侧是sentinel采样CPU占用，占用总CPU3%~4%，这部分应该是不会随流量上升而升高的， 这次就先没有动这块。第二块是在metaq的消费者代码里执行的，因此重点关注，因为我们的应用是消息驱动的，接受tp的发货消息后进行对应的操作，metaq流量升高，这部分对应的操作大概率也是会随之升高的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2n06wibxkOlRhGHKTicUwFPtSoadRicAWAz7gmebQSFzKCOia0LXOygBcwWg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2n2j8m999rP3CMibC85Ko5DwO72UMJ74JcicoQvXWUdWvlx7rkApTiaQmqQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

占总CPU占用达到了惊人的9.3%。

点开后可以看到是脱敏工具，其对性能的影响几乎和发货消息齐平了，排查后发现是我们部门内部使用的链路采集工具，在采集metaq消息时会对消息进行脱敏处理，脱敏工具会对姓名、邮箱、手机号等分别进行正则匹配，而我们接收的交易消息中是包含整个订单信息的，这个对象是很大的(包含扩展字段等诸多信息)，对其使用正则进行脱敏工作量巨大。正常情况下使用此工具采集线上流量对性能影响不是很大，但是在我们的场景影响有点出乎意料......

由于我们平时基本不会在链路图上关注消息的内容，一般都是用来看HSF链路，因此直接把dp对metaq的采集关闭了。

#### **案例二**

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2nBia0pXS9Q6Xf70ibb3YPgDezvfGVdw4icJzbCR8tAnoWyfhEY1cGMbTAQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2n93Qom9C8WDdDFdHicysDvKZd30gJw3rceeFpe7A5NeAlBeqgTyRibn0A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2nQm6etRiaOHDBKCP3gQT0dqxCkqjTjzibhAdxyJHhj0nCsDHDOfMRAicJA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用全局搜索后居然占用了将近6%的CPU，这可是日常的流量下截取的火焰图，系统流量升高时占用比例会更大。

点开后可以看到是进行HSF调用的时候，获取Java调用栈比较耗时，之前写代码的时候怀疑过获取调用栈会比较耗性能，但没想到居然比一次HSF调用本身的耗时都大了。这个地方之前是为了获取调用来源，打印到日志中方便排查问题的(历史代码)，后续将HSF的调用日志改成了通过HSF Filter的方式，去掉了获取调用栈的逻辑。

其实案例二我一开始不是从上往下看定位到的，因为调用HSF的地方不止一个，每个耗时其实也不长，整体看的话从上往下还是比较难发现的。

**从下往上看**

#### **案例二**

如果从上往下看效果不明显，可以从我们系统主要流量入口处进行分析，从下面入口一步步往上点，也可以发现问题，带着怀疑的态度来找，上述的案例二我一开始并没有注意到上方的问题，我是一步步从消息入口看，然后点到上面发现的这个调用消耗居然比HSF多的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2nFAgriadntyBFTkn9KR5jlOIkOECIN9iaXYicfMbAsLic62HziadIpfrYIFQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到这个调用不是集中在一个地方的，细看每个地方都有调用，所以整体对性能的影响才那么大。

全局搜索后可以看到到处都有调用(图示紫色部分)。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2nVk6KhXZYqQyiasR46XD3JYfpXOKXvWuv3fHP8kJUDBHBjFHB1Pqic0kA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从下往上找更适合细化的时候，专门对某个链路进行分析优化。

优化效果

下面来让我们计算一下数据，来看看楼主是不是标题党了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naI98zFLLVH60XmoQM34xt2nmZ4fU3viceicoiaHicoicgaQuzdPxKG8RqvNf5o0Nl5y29M1PQBoRQC9tvA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

整体大致优化了5~6%左右(7月份的时候，机器数量是30台，8月缩容到了27台)，取5%，其中系统CPU占用在6.5%左右，从日常流量上来看，用户态cpu从26%到21%，下降19%+，考虑缩容的关系，20%的优化大抵是有的👻。

由于这是日常流量的优化结果，大促流量突增时，系统负载降低应该会更明显，优化后大用户批量操作时瞬时流量也基本不会有机器cpu占用超过60%了。

后续双十一压测也会继续关注优化，流量上升后更多问题可能就会暴露出来。