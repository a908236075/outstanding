# docker 学习

## 常用docker命令

~~~shell
docker search 镜像名称 //搜索镜像
docker pull 镜像名称:版本号 //拉取对应版本的镜像
docker pull 镜像名称 //默认拉取最新的镜像
docker images //查看本地已下载的镜像
docker ps //查看正在运行的容器
docker ps -a //查看所有的容器（包括run、stop、exited状态的）
docker container ls //查看正在运行的容器
docker rm 容器ID //只能删除没有在运行的容器
docker rm -f 容器ID //可以删除正在运行的容器
docker rmi -f $(docker images -qa) 删除全部也可删除多个
docker run -p 本地主机端口号:容器服务端口号 --name 容器名字 [-e 配置信息修改] -d 镜像名字   // -d表示后台启动
docker start 容器ID //启动容器
docker stop 容器ID //终止容器
docker rmi 镜像名称orID //删除镜像
docker exec -it topisa /bin/bash  // 进入容器内部  可以更改欢迎页等信息
docker logs -f  容器ID  // 查看容器日志
docker update 容器ID --restart=always  // 更新容器的启动策略  随主机一起启动
##查询端口号占用
ss -antulp | grep POR 看下监听端口或者ps看见jboss相关进程
##添加端口对外映射
#查看端口映射列表
sudo iptables -t nat -vnL DOCKER --line-number
#端口映射 10902--->10902
sudo iptables -t nat -A  DOCKER -p tcp --dport 10902 -j DNAT --to-destination 172.17.0.2:10902
#删除规则 3
sudo iptables -t nat -D DOCKER 3
~~~

1. ![](..\typora\picture\云原生\docker 常用命令.png)

   ## docker 拷贝文件

   1. ~~~shell
      ##本地到docker
      docker cp /Users/howey/Documents/apache-maven-3.5.2/ 749056ea1637:/opt
      docker cp 本地路径 容器Id或name:容器目录
       
      ~~~

   2. ~~~shell
      ## docker 到本地
      docker cp testtomcat：/usr/local/tomcat/webapps/test/js/test.js /opt
      docker cp 容器Id:容器路径 宿主机路径
      ~~~

   ## 镜像操作

   1. 导入导出**容器**

      ~~~shell
      docker exprot -0 xx.tar  容器ID   // 导出镜像成为xx.tar
      docker import xx.tar 容器名称     // 导入镜像 导入的镜像不能直接启动 需要知道之前的启动的命令
      ## 启导入的容器 查看原容器的启动命令
      docker ps --no-trunc 查看原容器的启动命令 
      CONTAINER ID   IMAGE                                             COMMAND                  CREATED        STATUS       PORTS   
      cfx07xxx5      tcr/mqtt:20211119   "/usr/bin/docker-entrypoint.sh /opt/emqx/bin/emqx foreground"                                                   6 weeks ago    Up 6 weeks   4369-4370/tcp, 5369/tcp, 6369-6370/tcp, 8081/tcp, 0.0.0.0:1883->1883/tcp, 8083-8084/tcp, 0.0.0.0:8883->8883/tcp, 0.0.0.0:18083->18083/tcp, 11883/tcp   mymqtt
      docker run -p 本地主机端口号:容器服务端口号 --name 容器名字 [-e 配置信息修改] -d 镜像名字   
      ##或者用insepct命令查看原容器 查看EntryPoint+cmd 组合在一起.注意去掉符号
       "Cmd": [
                      "/opt/emqx/bin/emqx",
                      "foreground"
                  ],
      "Entrypoint": [
                      "/usr/bin/docker-entrypoint.sh"
                  ]
      ~~~

   2. 导入导出**镜像**

      ~~~shell
      docker save -o xxx.tar 镜像名称
      docker load -i xxx.tar   
      ~~~

## 核心原理

### 底层存储

1. 镜像为容器提供了基础的文件系统,不论容器是否在运行,关联的镜像都不能够删除.

2. 查看文件目录

   1. ~~~shell
       ## 注意是image 
       docker image inspect d79ac9e8379a
      ~~~

   2. ~~~json
       "GraphDriver": {
                  "Data": {
                      "LowerDir": "/var/lib/docker/overlay2/632be23db7f18f8b388512db39ee25e9814a859cba6e80c5a7c6e191cd46dfd7/diff:/var/lib/docker/overlay2/a75597a90558548c5cc723ffb4189ea77c7d0e39fbb35b545dc675bd931387eb/diff:/var/lib/docker/overlay2/43d8145bc4ca9442a718d01efa899e8b6483f3fd3d6a59b853234695d246eb4e/diff:/var/lib/docker/overlay2/2a30eebf60bb0b3364d2d26b8a92d1d34ff254bcf2a7900c877690d2ae336e09/diff:/var/lib/docker/overlay2/7b57ebd1120a0f49e8b54d20851abb993af47cda458ba81f5ff3f989e72189e1/diff:/var/lib/docker/overlay2/a9552bbc8d1a826da1027c34fb121dd623dac02f2771fe4c6c6e0e8725851b9c/diff:/var/lib/docker/overlay2/b16ff6b7911e9510ca3e982a4a94873e9bb188f63f3b6834ba689f70c748951e/diff",
                      "MergedDir": "/var/lib/docker/overlay2/a8b9f5ea7944eeb86d7f3b3ce0185997a42857a134c8f62f1c72158e206f5c70/merged",
                      "UpperDir": "/var/lib/docker/overlay2/a8b9f5ea7944eeb86d7f3b3ce0185997a42857a134c8f62f1c72158e206f5c70/diff",
                      "WorkDir": "/var/lib/docker/overlay2/a8b9f5ea7944eeb86d7f3b3ce0185997a42857a134c8f62f1c72158e206f5c70/work"
                  },
                  "Name": "overlay2"
              },
      ~~~

   3. 

3. 目录分类

   - 底层目录(镜像的目录)
   - 合并目录(底层目录和上层目录的合并,通过连接建立,是逻辑层不是物理层)
   - 上层目录(容器的目录,相较于底层目录变化的目录)
   - 工作目录

4. 容器共用镜像的底层文件.使用docker ps -s 可以看到容器真正使用的文件.镜像内的内容永远都不变.如果有修改就使用copyOnWrite到容器内.

5. 文件存储驱动

   1. CentOS overlayFS存储驱动.
   2. ![](..\typora\picture\云原生\docker_overlay_constructs.jpg)

6. 数据卷的挂载

   1. docker自动创建文件夹,然后将容器的内部的内容进行挂载指定文件夹.
   2. 自己创建文件夹,手动挂载.
   3. 数据挂载到内存. 一般不用
   4. 命令
      1.  -v 宿主机绝对路径:Docker 容器内部绝对路径 (具名卷)  ---->挂载
      2.  -v 不以/开头的路径:Docker 容器内部绝对路径(自动管理,不会把他当做目录,而是卷)  ----> 绑定
      3.  -v Docker 容器内部绝对路径 (匿名卷)
      4. 查看卷:docker volume 

7. DockerFile构建镜像

   1. 基础构建

      **FROM** 指定基础镜像 是什么环境就用什么

      **LABLE** 标签

      **RUN** 容器构建时候执行的命令

      **CMD** 容器启动的时候执行的命令

   2. ARG

      1. 先定义后使用
      2. CMD和ENTRYPOINT都是打印运行时候的变量.

   3. ENV

      1. ~~~dockerfile
         ENV version=1
         ~~~

      2. 构建期和运行期都能生效.

      3. 构建期不能改EVN的值

      4. 运行时候修改

         1. ~~~shell
            docker run -it -e version=2
            ~~~

      5. 持久化问题

         1. evn变量在构建的时候已经在容器的配置文件中持久化.运行时修改只能够改ENV本身,涉及到的引用不能修改.

         2. ~~~dockerfile
            ENV msg1=hello
            ENV msg2=${msg1}
            CMD["/bin/sh","-c","echo ${msg1};echo ${msg2};"]
            ## 运行时修改msg1 只能msg1生效  msg2由于读取的事持久化在文件中的变量的值,所以不会改变.
            ~~~

      6. 构建命令

         1. ~~~shell
            docker build --no-chache -t demo:test -f Dockerfile
            ~~~

   4. ADD和COPY命令

      1. ~~~dockerfile
         ADD https:://download.redis.xxx.tar.gz /dest/   # 从网址上下载 到/dest目录 
         
         Run cd /dest && ls -l    # 注意 RUN命令不是上下文的关系 如果两个命令有关联用&&连接  
         ~~~

      2. COPY 不会自动解压和下载 是从宿主机上复制

         1. ~~~dockerfile
            USER root:root    # 后面的命令使用root用户来运行.
            USER 1000:1000    # 后面的命令使用1000编号对应的用户来运行.
            ~~~

      3. WORKDIR

         1. WORKDIR为下面的命令指定目录.可以嵌套.
         2. 当进入容器控制台时候,直接进入指定目录.

      4. VOLUME 

         1. ~~~dockerfile
            VOLUME ["/hello","/app"] 
            ~~~

         2. 即使没有指定-v参数,也会匿名自动挂载.

         3. 已经挂载的目录,之后在对目录操作(例如:目录下文件内容的更改),是无效的.先修改在挂载.

      5. EXPOSE 

      6. ENTRYPOINT和CMD

         1. ENTRYPOINT 是启动命令 一般都是固定的.

         2. 都会覆盖,最后一个生效.

         3. ~~~dockerfile
            FROM alpine
            ENV url=baidu.com
            CMD ["/bin/sh","-c","ping ${url}"]  ## -c 作为一整串字符串, 推荐这种写法.
            ~~~

         4. ENTRYPOINT 是容器启动的唯一入口,CMD提供参数.如果想输入参数,整个CMD都会覆盖.

      7.  多阶段构建

         1. ~~~dockerfile
            FROM alpine AS buildapp
            ## 把上一个阶段的东西复制过来
            COPY --from=buildapp /app.jar /app.jar
            ~~~


## DockerFile

1. 瘦身

   1. run 可以使用&&合并在一起写 
   2. 添加.dockerignore文件
   3. 对阶段构建

2. touch命令

   1. 记录保存的时间点.

3. 网络

   1. 桥接网络.通过主机与docker0网关做转发.

   2. ~~~shell
      iptables -nL   ## 端口对应的映射关系  8007映射到 172.18.0.3 这个地址
      Chain DOCKER (2 references)
      target     prot opt source               destination         
      ACCEPT     tcp  --  0.0.0.0/0            172.18.0.2           tcp dpt:27017
      ACCEPT     tcp  --  0.0.0.0/0            172.18.0.3           tcp dpt:8007
      ACCEPT     tcp  --  0.0.0.0/0            172.17.0.2           tcp dpt:18083
      ACCEPT     tcp  --  0.0.0.0/0            172.17.0.2           tcp dpt:8883
      ACCEPT     tcp  --  0.0.0.0/0            172.17.0.2           tcp dpt:1883
      ~~~

   3. ~~~shell
       ip addr    ## ip列表
       docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
          link/ether xx:42:76:xx:76:98 brd ff:ff:ff:ff:ff:ff
          inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
             valid_lft forever preferred_lft forever
          inet6 fe80::42:76ff:feef:7698/64 scope link 
             valid_lft forever preferred_lft forever
      4: br-92b6bbcd9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
          link/ether 02:42:a1:86:62:8d brd ff:ff:ff:ff:ff:ff
          inet 172.18.0.1/16 brd 172.18.255.255 scope global br-92b69fd75c19
             valid_lft forever preferred_lft forever
          inet6 fexxx::42:a1ff:fe86:628d/64 scope link 
             valid_lft forever preferred_lft forever
      14: vethae1ddasa29@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-92b69fd75c19 state UP group default 
          link/ether 02:..80 brd ff:ff:ff:ff:ff:ff link-netnsid 0
          inet6 fe80::f:3eff:fe42:2280/64 scope link 
             valid_lft forever preferred_lft forever
      ~~~

   4. 

   5. 

