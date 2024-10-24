# 常用表达式

~~~sh
## 判断元数是否在集合中
name in {null,"Untitled"} 
## 获取集合的一部分数据
listeners.{? #this instanceof ActionListener}
## 获取静态变量
 @class@field
 ##获取Map中特定的key
getstatic com.yonyou.yuncai.cpu.service.dydp.AttributeToFactorUtil attributeToFactorcodeMap
getstatic com.yonyou.yuncai.cpu.service.dydp.AttributeToFactorUtil attributeToFactorcodeMap 'entrySet().iterator.{? #this.key.name=="billNum"}'
## 获取类中的静态变量 调用静态函数
watch demo.MathGame * '{params,@demo.MathGame@random.nextInt(100)}' -v -n 1 -x 2
[arthas@6527]$ watch demo.MathGame * '{params,@demo.MathGame@random.nextInt(100)}' -n 1 -x 2
## watch 方法中的复杂参数
watch com.yonyou.yuncai.cpu.service.dydp.BIDYDPCalculateServiceImpl executor "{params[0],returnObj,throwExp}" "params[0].get(0).formId=='PU.st_purchaseorder'" -x 4  -b
~~~

