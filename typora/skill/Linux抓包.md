# TCPDUMP命令

1. 抓去指定ip并写入文件

   1. ~~~shell
      tcpdump -i any host 10.53.110.110 -w p1.pcap
      ~~~

2. 只抓去22端口

   1. ~~~shell
      tcpdump -i eth0 tcp port 22 ##只抓TCP，22端口的包，这里我们用nc来连一下另一台虚拟机的22端口 
      ~~~

3. 捕获主机192.168.56.210接收和发出的tcp协议的ssh的数据包

   ~~~shell
   tcpdump tcp port 22 and host 192.168.56.210
   ~~~

4. 监听主机192.168.56.1和192.168.56.210之间ip协议的80端口的且排除www.baidu.com通信的所有数据包：

   1. ~~~shell
      tcpdump ip dst 192.168.56.1 and src 192.168.56.210 and port 80 and host ! baidu.com
      ## 或者即not和！都是相同的取反的意思
      tcpdump ip dst 192.168.56.1 and src 192.168.56.210 and port 80 and host not www.baidu.com
      ~~~

