# docker 学习

1. 常用docker命令

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
   docker run -p 本地主机端口号:容器服务端口号 --name 容器名字 [-e 配置信息修改] -d 镜像名字
   docker start 容器ID //启动容器
   docker stop 容器ID //终止容器
   docker rmi 镜像名称orID //删除镜像
   ~~~

2. docker 拷贝文件

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

