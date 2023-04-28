## 主要参数

1. ~~~json
    {
           "id":"auth",
           "order":1,
           "predicates":[
               {
                   "args":{
                       "pattern":"/gatway/auth/**"
                   },
                   "name":"Path"
               }
           ],
           "filters":[
               {
                   "name":"StripPrefix",
                   "args":{
                       "_genkey_0":"1"
                   }
               }
           ],
           "uri":"lb://auth-system"
       }
       // 老版本的配置
   ~~~

   

2. 参数解释

   1. 版本3.0 比较老  name和arg成对的配置

   2. predicates 进行路径的匹配 

   3. "uri":"lb://auth-system"  进行路径的跳转

   4. StripPrefix 过滤器进行路径的截断

   5. Weight断言

      1. ~~~yaml
         server:
           port: 8086
         
         spring:
           cloud:
             gateway:
               routes:
                 - id: server
                   uri: http://localhost:8085
                   predicates:
                     - Path=/message
                     - Weight=group1, 75
                 - id: new_server
                   uri: http://localhost:8085
                   predicates:
                     - Path=/message
                     - Weight=group1, 25
                   filters:
                     - PrefixPath=/new
         ~~~

      2. `Weight`路由断言工厂接受两个参数：`group`和`weight`（`int`类型），权重是按组计算的（`group`参数需要相同）。如果请求网关的`/message`路径，会有`75%`的概率被路由到`http://localhost:8085/message`，`25%`的概率被路由到`http://localhost:8085/new/message`（`PrefixPath`是一种路由过滤器，它会将指定的路径以前缀的形式加在请求路径上，因此请求`/message`路径，会被路由到`/new/message`路径）。

