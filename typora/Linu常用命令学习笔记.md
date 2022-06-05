## Linux概念和常用命令学习

## 基础

1. Linux目录结构
   - bin 目录 存放的是常用的指令.
   - dev:管理设备,将设备映射成为文件.
   - etc:配置文件.
   - opt:给主机额外安装软件所摆放的目录.
   - /usr/local:给主机额外安装软件所用到的安装目录.
   - /var:存放不断扩充着的东西,例如日志.
2. Vi和vim编辑器
   1. 文本编辑器 vim是vi的升级版.
   2. 正常模式,插入模式,命令行模式.
   3. 拷贝命令:退出编辑命令:p 拷贝当前行 yy5  并按p,复制5行.
   4. 删除当前行 dd,删除当先向下5行 5dd
   5. 在文件中查找某个单词:/+关键词  按n查找下一个
   6. 设置和取消文件行号:set nu和set nonu.
   7. 文件最末行:G,文件首行gg.
   8. 在文件中输入内容,退出编辑模式**,按u撤销刚刚的编辑.**
   9. :行号      可以跳转到指定的地方.
3. 用户管理和组
   1. 创建一个用户并切换:
      1. useradd talent
      2. password talent
      3. su - talent
      4. exit  返回为原来的root用户.
   2. 创建一个用户并制定用户组,在home中查看并用名查看.
      1. groupadd wudang 
      2. useradd -g wudang zwj
      3. id zwj
   3. 与用户和用户组相关的文件:
      1. etc/group  ect/shadow  etc/passwd
4. 实用命令
   1. 运行级别  单用户模式下找回密码

   2. man 或者 help帮助命令. 例如:man pwd

   3. 文件目录类
      1. pwd 当前目录
      
      2. touch 创建一个空目录.
      
      3. cp -r 可以拷贝整个文件夹.如果有覆盖每个文件都会提示,为了不让他提示我们可以使用\cp -r 拷贝命令.
      
      4. rm -rf  不光可以删除文件还可以删除文件夹
      
      5. cat -n 文件 | more  : 适合读取小文件,以只读的形式读取文件,-n为显示行号,more的命令可以让你按按空格显示更多.
      
      6. more 文本过滤器  以页的形式进行显,适用于读取文件内容较多的.
      
      7. less 分屏查看内容 对于大文件显示更快.快捷键:回车 下一行 空格下一页
      
      8. 由于more用上下键进行切换,所以用less实现了这个功能,less is more
      
      9. '>'输出重定向  >> 追加到文件中.">" 会覆盖原来的内容
   
      10. head  显示文件的开头内容. head -n 5 /etc/profile   默认10行
      
      11. tail  显示文件的后面的内容 默认为10行 tail -f 文件  时时监控文件所有的更新.
      
          1. ~~~shell
             tail -n 100 -f copyAndWrite.sh
             ~~~
      
      12. history  查看历史命令 !+命令号 直接执行对应的命令.
      
   4. 时间日期类
      
      1. date 显示时间  date -s 字符串时间     设置系统的时间
      
   5. 搜索查找类
   
      1. find **路径** 名称/size/user  查找文件的名称/大小/用户   例如 find /home -name "*.sh"
         1. 常用为-name 和-size
   2. grep  管道命令  cat b.txt | grep -n  a
   
6. 压缩文件命令
   
      1. gzip/gunzip  不会保留原文件的解压. 只能压缩文件,不能压缩目录.
      2. zip/unzip     zip -r mypackage.zip /home/    unzip -d /opt/tmp/  mypackage.zip
   3. tar打包命令  tar  -zcvf mytar.gz /home  将home打包到mytar.gz中. tar   -zxvf为解压命令. 
   
7. 组管理和权限管理
   
      1. ```shell
         -rw-r--r--   1 root     root     5.0 K 2月   9 08:50 bbb.txt
         ```
   ```
      
   2. -rw-r--r--:第一个符号为文件的类型:  -:普通的文件 d: 目录  l:链接  c:字符设备.
   
   3. rw-  文件所有者拥有的权限  r-- 文件所在组拥有的权限  r-- 文件的其它组拥有的权限.
   
   4. 1 如果是文件 表示硬链接的数  如果为目录, 表示该目录下子目录的个数.
   
   5. 5. 0 K  代表文件的大小.
   
   6. w:代表写的权限,但是不能删除文件,当有目录的写权限的时候,才会有删除文件的权限.
   
   7. 权限管理
   
         1. chmod命令  
      2. chown -R  tom  kkk\  将kkk下面的所有文件和目录所有者改成tom.
   ```
   
8. 任务调度
   
   1. crontab :不是复杂的任务可以不写脚本,直接用crontab.
   
9. 分区命令
   
      1. mount  挂载和卸载
      
      2. df  -l   显示磁盘的剩余情况
      
      3. du -l /目录   显示指定目录的情况
      
         1. du -h  /目录
      
            查询指定目录的磁盘占用情况，默认为当前目录
      
            -s 指定目录占用大小汇总
      
            -h 带计量单位
      
            -a 含文件
      
            --max-depth=1  子目录深度
      
         2. 
      
         ~~~shell
          du -ach --max-dept=1 /opt/
         0	/opt/container
         54M	/opt/kafka_2.11-2.0.0.tgz
         65M	/opt/kafka_2.11-2.0.0
         4.0K	/opt/tmp
         118M	/opt/
         118M	总用量
         ~~~
      
10. 进程相关的命令
   
       1. ps -aux
       2. ps -a:显示当前终端的所有进程信息.
       3. ps -u:以用户的格式显示进程信息.
       4. ps -x:显示后台进程运行的参数.
    5. 终止进程:
   
11. systemctl 服务命令
   
12. telnet 192.168.1.128:8080 查看服务的端口是否能通的命令.
   
13. 动态的监控进程
   
       1. top命令:动态的监视进程,与ps命令很相似.
    2. top命令每3s,查询一次,默认以cpu的使用率进行排序.M 设置内存使用率进行排序.N 以PID进行排序,q退出    k 输入想要杀死的进程.
   
14. 监控网络状态
   
       1. netstat -anp    	
          1. an:按照一定顺序输出.
       2. p:显示哪个进程被调用.
   
15. RPM包和YUM包
   
       1. RPM:RedHat Package  Manager:红帽包管理器.
       2. 检查rpm已经安装的列表:rpm -qa|grep xx

---

## shell 脚本

1. shell:是命令行解释器,向linux内核发送请求.

2. ~~~shell
   #!/bin/bash
   echo "hello,world!"
   ~~~

3. 可以用 sh helloword.sh 跳过执行权限直接执行.

4. vi中怎么赋值和移动行呢.

5. 定义变量的规则:

   1. 变量名可以由字母,数字和下划线,但是不能以数字开头.
   2. 等号两侧不能有空格.
   3. 变量名称一般习惯大写.

6. 定义后的环境变量需要刷新一下:source   /etc/profile

7. 多行注释为 :<<!  内容  !

8. 位置参数变量

   1. $ 可以直接输出接收到的参数  $1 是第一个参数,$*是所有的参数,$#参数的个数.$@也是所有的参数,不过作为单个的变量进行显示.

9. 预定义变量

   1. $$:获取当前进程号.
   2. $!:后台运行的最后一个进程的进程号.
   3. $?:最后一次执行的命令的返回状态.

10. 运算符

    1. RESULT=$[2*(2+3)] echo "result=$RESULT" 

11. 判断语句

    1. ```shell
       if [ 23 -gt 24 ]
       then 
       	echo "大于"
       fi	
       #注意空格!!!
       
       ```

    2. ~~~she
       [ ]里面写条件,then后加匹配了条件后要执行的语句,最后加上fi.
       ~~~

    3. ~~~shell
       3. ((  23  >  24 )) 等价于 [ 23 -gt 24 ]
       ~~~

    4. ~~~shell
       -e ##判断文件是否存在
           
              1. ~~~shell
                 if [ -e home/bbb.txt ]
                 then 
             	echo "存在!"
                 	fi
       ~~~
       ~~~

    5. ~~~shell
       ##常用的比较字符
              = 字符串比较
           
              -lt 小 于
           
              -le 小于等于
           
              -eq 等 于
           
              -gt 大 于
           
              -ge 大于等于
           
              -ne 不等于
           
              -f 文件存在并且是一个常规的文件
           
              -e 文件存在
           
              -d 文件存在并是一个目录
       ~~~

12. while 语句

    1. ~~~shell
       #计算小于输入值的整数和
       SUM=0
       i=0
       while [ $i -le $1 ]
       do
               SUM=$[$SUM+$i]
               i=$[$i+1]
       done
       echo "sum is $SUM"
       ~~~

13. for语句

    1. ~~~shell
       ## 100以内的整数和  
       SUM=0;
       for((i=0;i<=100;i++))
       do
               SUM=$[$SUM+$i]
       done
       echo "Sum is $SUM"
       ~~~

14. case 语句

    1. ~~~shell
       #打印星期
       case $1 in
       "1")
               echo "星期一"
       ;;
       "2")
               echo "星期二"
       ;;
       *)
               echo "其它"
       ;;
       esac
       ~~~

15. 工作中用到的语句

    1. ~~~shell
       #### 实际工作遇到的语句
       
       1. ~~~shell
          # 检查是否安装过mysql
          rpm -qa | grep mysql
          # 查询mysql位置
          whereis mysql
       ~~~
       
       2. 查看mysql 6612端口号是否被占用
       
          - ~~~shell
            ps -ef | grep 6612 | grep -v grep | awk 'NR==1 { print $2 }'
            # grep -v grep 去掉包含grep的进程行
            # awk 'NR==1 { print $2 }' 抽取第一行 打印第二列 即为占用此端口的pid
            #启动 mysql
            ./mysql.sh  start
            ~~~
       ~~~

---

# 鸟哥的私房菜

## 第二部分

1. ##### 文件与目录权限定义的不同

   1. 文件的-rwx权限,针对的是文件的内容.并不是文件本身.也就是说并**没有**删除文件的权限.
   2. 目录的-w权限,可以新建和删除文件.-x这里的执行权限代表的是可以**进入该目录的权限**.有时候用户有读的权限即知道目录结构,但cd进入不了.

2. 用bash和sh 执行shell时候,可以跳过权限认证. 即没有**执行**的权限也可以执行.

3. ~~~shell
   ## 查看系统剩余空间
   df -lh
   ~~~

4. 目录操作的相关命令

   1. ~~~shell
      mkdir -P test1\test2\test3
      rmdir test4ll ## 删除空文件 -p 可以删除多级空文件. 
      rm -r test ## 可以删除有内容的文件.默认加了提示内容 如果不想提示 就是加反斜杠
      \rm -r test
      ls v* ## 查找还有v开头的文件夹
      ~~~

   2. cd (change directory) 

   3. pwd(print working directory)

   4. 环境变量

      ~~~shell
      echo $PATH  ## 打印环境变量.
      PATH="${PATH}:/root" ## 将/root加入到环境变量
      ~~~

   5. recursive   递归的.

5. 文件内容的查看

   1. cat:从第一行显示文件内容:concatenate(串联).
   
   2. tac:cat的倒写,从最后一行查看.
   
   3. less
      - 空格键 向下饭一页
      - /字符串 向下查找关键字
      - ?字符串 向上查找关键字
      - n 重复前一个查找
      - N 反向重复前一个查找.
      - g 前进道文件的第一样
      - G 最后一行.
      - q 离开
   
   4. head
      - -n 后面接数字 表示显示几行  head -n 20 test.txt
   
   5. tail
      - tail -f test.log 持续显示文件末尾的数据.  f:follow  
      - tail -fn 20 test.log   -n 后面接显示的行数.
      - head -n 20 | tail -n 10 test.log 显示文件第10行到20行的数据.
   
   6. od 非纯文本文件查看
   
   7. touch 可以修改文件的时间,文件有三种时间:mtime:修改时间,ctime:状态时间(修改权限,状态改变),atime:读取时间.
   
   8. 默认权限
   
      1. 创建文件或目录**默认的权限**查询:umask -S 
   
         1. ~~~shell
            umask
            0002
            umask -S
            u=rwx,g=rwx,o=rx
            ~~~
   
         2. 0002 第一个不用管 后三位是表示去掉的权限.
   
         3. 文件与目录默认权限不同,文件默认就没有执行权限,在加上umask的去掉的权限.
   
   9. 用户可以修改文件的基本权限是什么:**具有目录的执行权限,具有文件的r和w的权限.**
   
   10. 

























































