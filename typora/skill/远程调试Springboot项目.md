## 远程调试Springboot项目及原理

1. ###  背景

   项目在本地正常运行,可是部署到服务器上面就会出现各种问题,这时候,需要知道线上的代码是怎么运行的.所以这个技能是每个java工程师必须get的.

2. ### 调试步骤

   - ##### 将项目打包,部署到服务器上,并使用对应虚拟机的参数启动jar包.

     ```she
     java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005 /home/test/teset.jar
     ```

     如果你的项目使用了项目内的jdk,而没有在环境中安装jdk,那需要使用本地内的jdk执行命令.

     ```shell
     /home/test/jdk/bin/java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005 /home/test/teset.jar
     ```

   - ##### 如果是Tomcat的工程,需要从服务器上复制tomcat的catalina.sh文件,在第一行添加参数,然后替换掉服务器上的该文件.

     - ```shell
       CATALINA_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8089"
       ```

     - **注意对应端口为8089,所以idea配置的时候,也要配成8089,而不再是5005了**

     - 配置后启动tomcat(startup.sh).

   - ##### 设置Idea,监听端口

     - 首先点击Edit Configurations

     ![image-20210125111946937](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20210125111946937.png)

     - 点击加号,添加Romote,然后编辑名称,连接参数等.

![image-20210125112953262](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20210125112953262.png)					

​	JVM的参数:

```xml
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

##### 	点击debug启动.idea开始监听5005端口.打断点,请求服务器接口,就能开始远程调试代码了!!

![image-20210125134614899](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20210125134614899.png)

3. ### 远程调试原理

   ​	本机和远程服务器的jvm使用Debug协议通过socket通信,传递调试指令和调试信息.即使用本地代码作为一个UI界面,通过protocol,调用远端的JVM进程.

4. ### 参数解读

   - jwdp:Java Debug Wire Protocol的缩写.
   - transport:调试的程序和JVM使用的进程之间的通讯.
   - dt_socket:套接字传输
   - server(y/n):VM是否需要作为调试服务器.
   - suspend(y/n):是否在调试的客户端建立连接后启动VM.
   - address:调试服务器监听的端口号.

### 注意事项

1. 服务器启动的时候可能会报Caused by: java.lang.ClassNotFoundException: org.springframework.scheduling.TaskScheduler 异常 不用管 ,其实服务已经启动了.

2. 查看防火墙状态 以及 关闭防火墙

   1. ~~~shell
      ## 查看防火墙状态
      systemctl status firewalld 
      ~~~

   2. ~~~shell
      ## 关闭/开启/重启 防火墙
      service firewalld stop/start/restart
      systemctl stop firewalld
~~~
      
      

## 实际项目远程debug

1.SpringBoot项目启动时加以下参数
	-Xdebug -Xrunjdwp:transport=dt_socket,address=5005,server=y,suspend=n

​	如：nohup java $jvm_options -Xdebug -Xrunjdwp:transport=dt_socket,address=5005,server=y,suspend=n -Dloader.path=lib,config -jar 		       ${BASE_PATH}/$jarPath --spring.config.location=${CONFIG_DIR},${com_path}/comom_config/application-mysql.yml,${com_path}/comom_config/application-es.yml -> /dev/null 2>&1&
2.关闭Linux防火墙
​	systemctl stop firewalld

3.Idea配置远程调试
​	新建Romote,输入相应的远程IP和端口（默认5005）

注：本地代码需与远程代码保持一致





