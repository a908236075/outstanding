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

   10. 磁盘:

       1. superblock:记录此filesystem 的整个信息,包括inode/block的总量,使用量,剩余量,以及文件系统的格式和相关信息等.
       2. inode 记录了文件的属性(拥有者,权限,最近修改的时间等),一个文件占用一个inode,同时记录此文件的数据所在的block号码.
       3. block:实际记录文件的内容,若文件太大时,会占用多个block.
       4. 如果文件很小,则剩余的block的空间是不能被使用的.也不能将block定义的太小,会导致大文件占用太多的block,影响读写的性能.

   11. 挂载点为目录,该目录为进入该文件系统的入口.

   12. df:列出文件系统的整体磁盘使用量.

       1. df -h 以易读的形式返回磁盘使用信息.

   13. du:评估文件系统的磁盘使用量.

       1. du 显示文件的容量  单位是KB.

   14. 创建目录快捷方式: ln -s  pwsswd-so 如果加-s就是symbol link.否则就是hard link.

   15. 磁盘分区

       1. lsblk :列出所有存储装置(list block device);

       2. ~~~shell
          [root@bogon /]# lsblk
          NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
          sda      8:0    0   20G  0 disk 
          ├─sda1   8:1    0  300M  0 part /boot
          ├─sda2   8:2    0  512M  0 part [SWAP]
          └─sda3   8:3    0 19.2G  0 part /
          sr0     11:0    1  4.5G  0 rom  
          #MAJ:MIN:核心认识的装置都是透过这两个代码来熟悉的!粉笔是主要的:次要的装置代码.
          #RM:是否可以卸载的装置,
          #RO:是否是只读装置.
          #TYPE:是磁盘(disk),分区盘(partition)还是只读存储器(rom)等输出.
          ~~~

       3. blkid:列出装置的UUID等参数.

       4. 透过blkid 也知道了所有的文件系统! 想要进一步知道磁盘的分区类型 parted这个命令

          1. ~~~shell
             [root@bogon dev]# parted /dev/sda print
             Model: VMware, VMware Virtual S (scsi)
             Disk /dev/sda: 21.5GB
             Sector size (logical/physical): 512B/512B
             Partition Table: msdos
             Disk Flags: 
             
             Number  Start   End     Size    Type     File system     标志
              1      1049kB  316MB   315MB   primary  xfs             启动
              2      316MB   852MB   537MB   primary  linux-swap(v1)
              3      852MB   21.5GB  20.6GB  primary  xfs
             ~~~

          2. Partition Table 分区表的格式 :msdos 微软磁盘操作系统,MBR:

       5. 磁盘分区

          1. ~~~shell
             [root@bogon dev]# fdisk /dev/sda
             欢迎使用 fdisk (util-linux 2.23.2)。
             
             更改将停留在内存中，直到您决定将更改写入磁盘。
             使用写入命令前请三思。
             
             
             命令(输入 m 获取帮助)：p
             
             磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
             Units = 扇区 of 1 * 512 = 512 bytes
             扇区大小(逻辑/物理)：512 字节 / 512 字节
             I/O 大小(最小/最佳)：512 字节 / 512 字节
             磁盘标签类型：dos
             磁盘标识符：0x000ba003
             
                设备 Boot      Start         End      Blocks   Id  System
             /dev/sda1   *        2048      616447      307200   83  Linux
             /dev/sda2          616448     1665023      524288   82  Linux swap / Solaris
             /dev/sda3         1665024    41943039    20139008   83  Linux
             ~~~

          2. 创建一个新的分区

             1. ~~~shell
                [root@localhost dev]# fdisk /dev/sda
                欢迎使用 fdisk (util-linux 2.23.2)。
                
                更改将停留在内存中，直到您决定将更改写入磁盘。
                使用写入命令前请三思。
                
                
                命令(输入 m 获取帮助)：n
                Partition type:
                   p   primary (2 primary, 0 extended, 2 free)
                   e   extended
                Select (default p): p
                分区号 (3,4，默认 3)：4
                No free sectors available
                
                命令(输入 m 获取帮助)：n       
                Partition type:
                   p   primary (2 primary, 0 extended, 2 free)
                   e   extended
                Select (default p): p
                分区号 (3,4，默认 3)：3
                ## 选择可用的分区和容量后打印磁盘信息
                命令(输入 m 获取帮助)：w
                命令(输入 m 获取帮助)：p 
                ~~~

             2. ~~~shell
                [root@localhost dev]# cat /proc/partitions 
                major minor  #blocks  name
                
                   8        0  209715200 sda
                   8        1     512000 sda1
                   8        2  209202176 sda2
                  11        0    4481024 sr0
                 253        0  205004800 dm-0
                 253        1    4194304 dm-1
                 ## 看到并没有我们新创建的分区 因为这时候分区核心并没有更新.
                 [root@bogon dev]# partprobe -s
                /dev/sda: msdos partitions 1 2 3
                Warning: 无法以读写方式打开 /dev/sr0 (只读文件系统)。/dev/sr0 已按照只读方式打开。
                /dev/sr0: msdos partitions 2
                ## 使用partprobe后进行刷新.
                ~~~

       6. 压缩技术:读取已byte为单位,为了方便读取需要填充0,压缩技术就是利用这些0.

       7. 文件与文件系统的压缩

          1. **压缩**:tar -jcv -f filename.tar.bz2  **要被压缩的文件或目录名称**
          2. **查询**:tar -jtv  -f filename.tar.bz2
          3. **解压缩**:tar -jxv -f  finename.tar.bz2 -C **欲解压的目录**
          4. 命令中的j标识支持bzip2 文件名最好命名为.bz2,如果换成z,则是支持zp格式. 例如tar zcv -f  /root/etc.tar.gz /etc.换成J则表示支持xz,文件名以xz结尾.
          5.  将**部分文件**进行**解压**
             1. tar -jtv -f /root/ext.tar.gz2 | grep 'shadow'  先查询想要解压的文件
             2. tar -jxv -f /root/ext.tar.gz2  /etc/shadow  文件就会加压到当前文件夹的etc/shadow中了
          6. 将**部分文件**进行**压缩**
             1.  tar -jvc -f /root/system.tar.bz2 --exclude=/root/etc*  /tmp/
             2. 压缩/tmp中除了/root/etc*的文件 压缩到/root/system.tar.bz2.

       8. vim程序编辑器

          1.  ctrl +u 向上移动半页
          2. ctrl + d 向下移动半页.
          3. nyy 复制光标所在的向下n列的.
          4. p 粘贴.
          5. ndd 删除光标所在的向下n列.
          6. G移动文档的最后一列.移动到某一行25G.
          7. gg 移动到文档的第一列.
          8. n <Enter> 光标向下移动n列.
          9. x,X x向后删除一个字符(相当于del) X 向前删除一个字符相当于backspace.
          10. u 复原前一个动作
          11. Ctrl + r 重复上一个动作. 
          12. :n1,n2s/word1/word2/g 在n1与n2列之间寻找word1 并将word1替换为word2
          13. :1,$s/word1/word2/gc 从第一列到最后一列寻找并替换.并一个一个单词的comfirem
          14. :sp 在打开的文档中输入:sp 打开多窗口. Ctrl+W+↑或者↓ 切换窗口.Ctrl+W+q 全部离开.
          15. set nu set nonu  设置行号 或者取消行号.

       9. 认识与学习bash

          1. type 命令 可以查看是否是内容的bash命令.例如 type cd

          2. ctrl+u 删除光标前面的命令,ctrl+k 删除光标后面的命令.

          3. **设置变量** 变量名=内容.变量名只能是英文字母或者数字,但是不能以数字开头.

          4. unset 变量名 **取消变量**.

          5. **双引号**引用的内容可以保有变量的内容(也就是会转换).**单引号**不会.

          6. set 观察所有的变量.unset name 可以取消变量名为name的变量.

          7. export 变量名 自定义变量转变成环境变量. 没有指定变量名就显示所有的**环境**变量.

          8. ulimit -a 查看所有限制用户的系统资源参数.例如最多打开多少文件数量.

          9. hsitory 10 显示最近10条的命令 !88 执行第88条命令 !! 执行上一条命令.

          10. $? 获取前一个命令的输出结果.

          11. 与环境变量相关的配置文件 略过

          12. 数据流重定向

              1. **标准输入:代码为0,使用<或<<**

              2. **标准输出:代码为1,使用>或>>**

              3. **标准错误输出:代码为2,使用2>或2>>;**

              4. ~~~shell
                 find /home/ -name .bashrc >list_right 2>list_error ## 将正确信息和错误信息分别输出
                 ~~~

              5. 如果想让正确的信息和错误的信息都写入到**同一个文件中**,注意写法

                 1. ~~~shell
                    find /home/ -name .bashrc >list_right 2>list_right ##错误!!!!
                    ~~~

                 2. ~~~shell
                    find /home/ -name .bashrc >list_right 2>&1 ## 正确
                    find /home/ -name .bashrc &>list_right  ## 正确
                    ~~~

              6. /dev/null垃圾桶黑洞设备与特殊写法

                 1. ~~~shell
                    find /home/ -name .bashrc 2>/dev/null
                    ~~~

              7. 输入符号 > 例如将 ll -d >1.txt 中 ,每次将旧的的数据覆盖掉,如果不想覆盖,使用衔接符号>>.

          13. 如果想多条命令一起执行,需要将命令用;分开 例如 sync;sync

          14. 命令有前因后果的关系:
              1. cmd1&&cmd2 当cmd1执行成功cmd2命令才会执行.
              2. cmd1||cmd2 当cmd1不执行成功了 cmd2才会执行. 

          15. 管道命令
              1. 管道命令仅会处理标准输出,对于标准错误会予以忽略.

              2. cut -d 后面接分个字符 -f 第几段      用分个字符将输入切分成几段 -f 后面接数组 取出第几段

                 1. ```shell
                    echo $PATH | cut -d ':' -f 5 ##/usr/local/java/jdk1.8.0_291/bin
                    ```

              3. grep 提取 -v 反向选取  

                 1. ```shell
                    last | grep -v "root"  ## 命令中不包含root的字符.
                    ```

              4. uniq 唯一显示

                 1. ~~~shell
                     cat /etc/passwd | sort -n | uniq -c ##-c 进行计数
                    ~~~

              5. wc [lwm] 查看文件的行数 字数和字符数.

              6. tee 将处理的数据直接写入到文件或者屏幕上.

                 1. ~~~shell
                    last | tee last.list | cut -d " " -f 1 ##只是将数据进行保存 不影响后续的操作.
                    ~~~

                 2. 不同于>或者>> 是将所有的输入都写入某个文件,tee可以继续处理,而不是终结输入.

              7. 有时候用到tee将内容写入新的文件中,需要对部分字段做处理,就要用到col(将tab转成空格键),join,paste,expand等关键字.

              8. split 将文件进行分隔.

                 1. ~~~shell
                    split -b 300k /etc/services services
                    ~~~

          16. 正则表达式与文件格式化处理

              1. ^ 在括号之内[ ]代表反向选择,在[ ]之外则代表定位在首行.

                 1. ~~~shell
                    grep -n '[^[:lower:]]' man_db.conf ## 表示显示包含除了a~z的行.
                    ~~~

              2. 在文件中,每一行的空格前都隐藏这$,正则表达^$ 代表以$开头的意思

                 1. ~~~shell
                    grep -v '^$' /etc/rsyslog.conf | grep -v '^#' ##  -v 是反选 ,表示去掉空格行和去掉注释行.每一个空行其实开头都有一个隐藏的$
                    ~~~

              3. *代表有任意个可以为0个,.代表一定有一个字符的意思.

                 1. ~~~shell
                    grep -n 'oo*' /etc/rsyslog.conf ##至少含有一个o的行.
                    ~~~

                 2. ~~~shell
                    grep -n 'g.*g' /etc/rsyslog.conf ## .* 代表0个或者多个任意字符,过滤了含有g......g的行数.
                    ~~~

              4. 限定连续RE字符范围{}

                 1. ~~~shell
                    grep -n 'go\{2,5\}g' regular_express.txt 
                    ## {} 需要使用\转义 表示g后面接2-5个o的行.
                    ~~~

              5. 正则表达式与一般在命令行输入的通配符并不相同

                 1. ~~~shell
                    ls -l a* 与 ls | grep -n '^a.*' ## 此语句都表示显示以a开头的文件 
                    ~~~

              6. sed具有比grep更强大的更改的功能,以后才了解.

              7. 扩展正则表达式

                 1. 重复一个或者一个以上的前一个RE字符. --> + 例如 grep -n 'go+d' 1.txt
                 2. 零个或者一个 ---> ?
                 3. 用或(or) 的方式找出数个字符串 --->  | 
                 4. 找出字符串群组  ----> ( )

              8. awk 主要处理每一行的字段内的数据,而默认的字段的分隔符为"空格键或者Tab键".

                 1. ~~~shell
                    last -n 5 | awk '{print $1 "\t" $3}'
                    ~~~

                 2. awk可以使用<,>等,还可以做计算.

              9. 文件对比工具.

                 1. diff 比较文本内容的行.
                 2. cmp 主要用于比较二进制文件.
                 3. patch 可以将旧数据更新到新的版本中.

          17. shell script 学习

              1. 执行shell 需要具有r和x的权限,如果是sh 1.shell 这种方式执行,就只需要有r的权限即可.

              2. 练习

                 1. ~~~shell
                    #!/bin/bash
                    ## 创建三个文件 与当前的日志联动
                    read -p "Please input your filename: " fileuser
                    filename=${fileuser}
                    date1=$(date --date='2 days ago' +%Y%m%d)
                    
                    date2=$(date --date='1 days ago' +%Y%m%d)
                    date3=$(date  +%Y%m%d)
                    file1=${filename}${date1}
                    file2=${filename}${date2}
                    file3=${filename}${date3}
                    touch "${file1}"
                    touch "${file2}"
                    touch "${file3}"
                    ~~~

                 2. ~~~shell
                    #!/bin/bash
                    ## 乘法计算
                    read -p "Please input firstNum : " firstNum
                    read -p "Please input secondNum : " secondNum
                    total=$((${firstNum}*${secondNum}))
                    echo -e "\n The result ${firstNum} * ${secondNum} is ===> ${total}"
                    ~~~
                   
                 3. $((计算表达式))    echo $((13*6))

              3. $() - 命令执行结果替换

                 1. ~~~shell
                    echo Today $(echo is $(date "+%Y-%m-%d"))
                    ~~~

                 2. 执行$(date "+%Y-%m-%d")括号中的date "+%Y-%m-%d"，结果为2020-05-01
                    替换$(date "+%Y-%m-%d")为执行结果2020-05-01，命令还剩下echo Today $(echo is 2020-05-01)
                    同理执行$(echo is 2020-05-01)并替换，命令还剩下echo Today is 2020-05-01
                    最终输出Today is 2020-05-01

              4. 使用sources 执行命令,使子线程的变量在父线程中生效.

              5. 使用test进行判断

                 1. ~~~shell
                    #!/bin/bash
                    ## 判断文件权限
                    read -p "Please input filename" filename
                    test -z ${filename} && echo "filename is null"&& exit 0
                    test ! -e ${filename} && echo "The filename "${filename}" not exist" && exit 0
                    test -f ${filename} && filetype="regular file"
                    test -d ${filename} && filetype="directory"
                    test -r ${filename} && perm="readable"
                    test -w ${filename} && perm="${perm}writable"
                    test -x ${filename} && perm="${perm}executable"
                    echo "The filename: ${filename} is a ${filetype}"
                    echo "And the permission for you are : ${perm}"
                    ~~~

              6. 使用[ ] 判断  括号两边需要有空格  

                 1. ~~~shell
                     [ "${name}" == "tom cat" ] && echo "Yes" ## ${name} 需要用" "括起来.不然不会作为一个整体进行比较.
                    ~~~

              7. sh后的参数的序号

                 1. ~~~shell
                    /path/to/scriptname  opt1    opt2  opt3   opt4
                    		${0}         ${1}    ${2}   ${3}   ${4} 
                    ~~~

                 2. 可以在.shell文件通过$1 获取第一个参数.

                 3. shift 可以拿掉之前的参数  shift 3 去除掉三个参数.

              8. 条件判断

                 1. ~~~shell
                    if [空格 条件1 空格]; then
                    	当条件1成立执行的命令
                    elif [空格 条件2 空格]; then
                    	当条件2成立执行的命令
                      else
                    	最后执行的命令
                    fi
                    ~~~

                 2. ~~~shell
                    #!/bin/bash
                    ## 注意 test 与 [ ] 判断直接的差别
                    if [ "${1}" == "hello" ]; then
                            echo "Hello,how are you"
                    elif [ "${1}" == "" ]; then
                            echo "please input param"
                    else
                            echo "please input hello"
                    fi
                    ~~~

                 3. test 一般是命令输入的形式 [ -z "${1}" ] 等价于 test -z "${1}",可以把[ ] 看做是test

                 4. netstat -tuln 可以查看目前系统开放了哪些端口,0.0.0.0 是面向网络的.

                 5. ~~~shell
                       #!/bin/bash
                       echo -e "The ftp,ssh will be detect!!"
                       testfile=/home/bin/checking.txt
                       netstat -tuln > ${testfile}
                       testing=$(grep ":22 " "${testfile}")
                       if [ "${testing}" != "" ]; then
                       echo -e "SSH is running in your system!!"
                       fi
                       ## 遇到了错误有
                       ## 1.testfile 如果不存在会创建的 不用单独判断
                       ## 2.${testfile} 在使用的时候需要加"" 这才才会是环境变量进行转换
                       ## 3.if 空格[空格 判断 空格] 忘记加空格
                    ~~~

                 6. ~~~shell
                    #!/bin/bash
                    read -p "Please input date: " startdate
                    date1=$(echo "${startdate}")
                    echo "${date1}"
                    declare -i date_dem=$(date --date="${date1}" +%s)
                    declare -i date_now=$(date +%s)
                    declare -i date_total_s=$((${date_dem}-${date_now}))
                    declare -i date_total_d=$((${date_total_s}/60/60/24))
                    if [ "${date_total_d}" -lt "0" ]; then
                    echo "You had been  before: " $((-1*${date_total_d}))
                    else
                    echo "You will after: ${date_total_d} days "
                    fi
                    ## 遇到的问题
                    ## 1.计算公式中的变量 不需要用" " 括起来 这和 [ ] if条件判断不太一样
                    ~~~

              9. case .... esac 判断

                 1. ~~~shell
                    #!/bin/bash
                    case ${1} in ##参数 in
                    "hello")
                            echo "hello! How are you?"
                            ;;  ## 结束 必须要有
                    "")
                            echo "Please input your params,ex> {${0} someword}"
                            ;;
                    *)     echo "Use  ${0} {hello}" ##其余都不匹配的时候
                    esac
                    ~~~

              10. 利用 function功能

                  1. 由于shell 执行的顺序是**从上到下 从左到右** 所以function需要先定义,才能使用.
                  2. function fname(){ 程序段 }
                  3. function中也有内置变量,$0表示function名 $1表示第一个变量,在调用function时候,需要区分是函数的$1,还是执行shell 的第一个参数

              11. 循环

                  1. while do done,until do done 不定循环

                     - ~~~shell
                       #!/bin/bash
                       while [ "${ny}" != "yes" -o "${ny}" != "YES" ]
                       do
                               read -p "Please input 'yes' or 'YES' to intterupt running " ny
                       done
                               echo "your input is right!!"
                       ## 遇到的问题
                       ## 1.while的[空格 判断条件 空格] 忘记加空格
                       ## 2.[空格 判断条件 空格] 判断条件的 yes 用 " "包含进来. 
                       ~~~

                     - ~~~shell
                       #!/bin/bash
                       until [ "${ny}" == "yes" -o "${ny}" == "YES" ]
                       do
                               read -p "Please input 'yes' or 'YES' to intterupt running " ny
                       done
                               echo "your input is right!!"
                       
                       ~~~

                  2. for...do ..done (固定循环)
                  
                     1. ~~~shell
                        #!/bin/bash
                        users=$(cut -d ":" -f1 /etc/passwd)
                        ##echo "${users}"
                        for user in ${users}
                        do
                                id ${user}
                        done
                        ~~~
                  
                     2. ~~~shell
                        #!/bin/bash
                        for animal in dog cat elephant
                        do
                                echo "the zoo hava animal : ${animal}"
                        done
                        ~~~

          18. Linux账号管理

              1. 查询账号各参数的意思 略

              2. id 命令 查询对应用户的所属分组等信息  id testuser

              3. 用户管理

                 1. ~~~shell
                    #添加用户
                    ## useradd [-u UID 即用户标识] [-g 初始用户组] [-G 次要用户组 账号还可以加入的用户组] [-s shell 例如nolgin shell]
                    useradd -g users testuser
                    grep testgroup /etc/group /etc/shadow # 查看是否创建成功
                    #修改用户
                    usermod -e "2022-06-22" testuser ## 修改用户失效时间
                    #删除用户
                    userdel -r testuser  ## 连home目录一起删除.
                    ~~~

              4. 用户组管理

                 1. ~~~shell
                    #添加用户组
                    ## groupadd [-g gid] [-r 建立系统用户组] 用户组名称 用户组名
                    groupadd testgroup
                    #修改用户组
                    ##groupmod [-g gid] [-n goup_name 修改既有用户组名] 用户组名
                    groupmod -n mygroup testgroup # 将testgroup 修改为mygroup 注意 -n 后面接的是修改后的名字.
                    #删除用户组
                    groupdel mygroup
                    ~~~

          19. 磁盘配额(Quata)与进阶文件系统管理

              1. 磁盘配额一般会针对用户,用户组,路径进行限制.

              2. 磁盘配额使用限制

                 1. 在ETX文件系统中仅能针对整个filesystem.
                 2. 核心必须支持磁盘配额(Quata)
                 3. 只对一般身份使用有效,root是不能够进行限制的,因为它拥有所有资源.
                 4. 若启用了SELinux,非所有目录均可设定quata,SELinux 防止用户误操作磁盘,如果想设置需要先关闭它.

              3. 实战操作

                 1. ~~~shell
                    ## 查看文件系统类型 是xfs还是etx
                    [root@bogon ~]# df -hT /home/
                    文件系统       类型  容量  已用  可用 已用% 挂载点
                    /dev/sda3      xfs    20G  4.8G   15G   25% /
                    ~~~

              4. 磁盘配额的应用与实践

                 1. xfs_quota -x -x 相关命令可以创建分区 将更具用户组和用户以及路径设置分区权限等.

                 2. LVM 逻辑卷管理器 
                 
                 3. **磁盘,分区与文件系统:**磁盘的文件一般是/dev/sd[a-p],磁盘又会像window中分C盘,D盘,在linux 中叫做分区,分区的格式有MBR和GPT两种,在分区操作之前,需要格式化成固定的文件系统,因为不同的文件系统权限和属性不同,常见的文件系统格式有ext2,xfs,ntfs,fat等. 传统的磁盘一个分区只能被格式化成一种文件系统.先进的则不同,所以一个可挂载的数据为一个**文件系统**而不是一个文件分区.
                 
                 4. **文件系统:**文件系统通常将文件属性和权限放在inode中,而数据则分开放在数据区块中.文件系统被分了好多inode.
                 
                 5. **挂载点**:每个文件系统都有独立inode,区块,超级区块等信息,文件系统能够连接到**目录树**,才能被我们使用,**挂载点一定要是目录,改目录进入该文件系统的入口.**
                 
                 6. ext文件系统使用命令 dumpe2fs 查询inode等系统信息,而xfs使用xfs_info.
                 
                 7. df 列出文件系统的**整体**磁盘使用量. 常用命令为 df -h
                 
                 8. du 查看文件系统的磁盘使用量.
                 
                 9. 观察磁盘分区的状态
                 
                    1. ~~~shell
                       lsblk ## 列出系统所有的磁盘与磁盘内的分区信息.
                       NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
                       sda               8:0    0   30G  0 disk
                       ├─sda1            8:1    0  512M  0 part /boot
                       └─sda2            8:2    0 29.5G  0 part
                       ├─centos-root 253:0    0 28.5G  0 lvm  /
                       └─centos-swap 253:1    0    1G  0 lvm  [SWAP]
                       sr0              11:0    1 1024M  0 rom
                       ## disk 磁盘  part 分区 
                       ~~~
                 
                    2. ~~~shell
                       blkid ##列出设备的UUID  
                       ~~~
                 
                    3. ~~~shell
                       parted /dev/sda print ## 列出磁盘的相关信息
                       Model: VMware, VMware Virtual S (scsi)
                       Disk /dev/sda: 32.2GB
                       Sector size (logical/physical): 512B/512B
                       Partition Table: msdos ## 分区格式是MBR,用fdisk命令
                       Disk Flags:
                                          
                       Number  Start   End     Size    Type     File system  Flags
                       1      1049kB  538MB   537MB   primary  xfs          boot
                       2      538MB   32.2GB  31.7GB  primary               lvm
                       ## 非常重要
                       ## Partition Table: msdos msdos 分区格式是MBR,用fdisk命令,gpt分区格式是GPT,用gdisk命令!!!
                       ~~~

          20. 进程管理与SELiuux 初探

              1. 进程ID:系统只认识二进制文件,每一次执行任务都会触发/bin/bash这个二进制文件去执行任务,根据UID/GID生成PID,用来标识进程.

               2. **程序**是二进制文件,一般存放在磁盘中,**进程**是将程序的代码和所需要的数据加载到内存中,并生成PID,是正在运行的程序.

               3. 一般的命令例如touch 执行后就会结束进程,但是向crontab有关的要一直查看是否任务需要执行,一直要占用进程,常驻在内存当中这种称为**服务**(deamon);

               4. 后台执行 在命令后面加一个&  使其在后台执行.

                  1. ~~~shell
                     tar -zpcf /tmp/1.gz /etc/ &
                     ~~~

               5. 把当前工作丢入背景中: Ctrl + Z  

               6. jobs 查看丢入背景中工作,+代表最近被放入到背景 的工作号码,-代表第二个.

                  1. ~~~shell
                     [root@bogon bin]# jobs -l
                     [1]-   305 停止                  vi hello.sh
                     [2]+  3102 停止                  vi hello-2.sh
                     ~~~

                  2. ~~~shell
                     fg %1  ## 取第一个放入背景中的job
                     ~~~

                  3. ~~~shell
                     [root@bogon bin]# jobs
                     [2]+  已停止               vi hello-2.sh
                     [root@bogon bin]# kill -9 %2; jobs    ### 杀死后台中的工作
                     [2]+  已停止               vi hello-2.sh
                     ~~~

               7. nohup 命令可以是任务在脱机的情况下依然执行.

               8. ps -l  仅观察自己的bash相关进程.

                  1. ~~~shell
                     [root@bogon bin]# ps -l
                     F S   UID    PID   PPID  C PRI NI ADDR SZ WCHAN        TTY      TIME     CMD
                     0 R     0   4730 128757  0  80    0 - 38329 -          pts/2    00:00:00 ps
                     4 S     0 126134 126114  0  80    0 - 28970 do_wai     pts/2    00:00:00 bash
                     4 S     0 128757 126134  0  80    0 - 28970 do_wai     pts/2    00:00:00 bash
                     # F 进程旗标 说明进程的总结权限 4 为root 权限.
                     # S 代表进程的状态   R(Running) S(Sleep) Z 僵尸进程.
                     # PRI/NI CUP所执行的优先级 越小越优先.
                     # ADDR SZ WCHAN 都与内存有关
                     # TTY 登入者的终端位置 
                     # TIME 耗费CUP的时间
                     # CMD 命令
                     ~~~

               9. ps  aux 显示系统所有的进程

                  1. ~~~shell
                     root@bogon bin]# ps aux
                     USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
                     root          1  0.0  0.3  51732  3096 ?        Ss   6月27   0:41 /usr/lib/systemd/systemd --switched-root --syste
                     root          2  0.0  0.0      0     0 ?        S    6月27   0:00 [kthreadd]
                     # STAT 状态 
                     # COMMAND 命令
                     root@bogon bin]# ps axjf   树形结构显示进程  
                     PPID    PID   PGID    SID TTY          TPGID STAT   UID   TIME COMMAND
                          1    849    849    849 ?            -1 Ss       0   0:00 /usr/sbin/sshd -D
                        849 126114 126114 126114 ?            -1 Ss       0   0:00  \_ sshd: root@pts/2
                     126114 126134 126134 126134 pts/2      5447 Ss       0   0:00      \_ -bash
                     126134 128757 128757 126134 pts/2      5447 S        0   0:00          \_ bash
                     128757   5447   5447 126134 pts/2      5447 R+       0   0:00              \_ ps axjf
                     ~~~

               10. top命令

                   1. ~~~shell
                      top -d 2 ## 每两秒更新一次top,观察整体的信息.
                      top - 17:39:51 up 3 days, 17:06,  2 users,  load average: 0.03, 0.04, 0.05
                      Tasks: 106 total,   2 running, 104 sleeping,   0 stopped,   0 zombie
                      %Cpu(s):   0.0/0.0     0[                                                                                        ]
                      KiB Mem :   995684 total,    70760 free,   313748 used,   611176 buff/cache
                      KiB Swap:   524284 total,   401148 free,   123136 used.   488896 avail Mem 
                      
                      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                      
                      285 root      20   0       0      0      0 S  0.5  0.0   1:58.20 xfsaild/sda3                                  
                        1 root      20   0   51732   3096   2004 S  0.0  0.3   0:42.00 systemd 
                      # PR 优先级 越小越好
                      # TIME+ 累计占用cpu 的时间.
                      ~~~

                   2. top命令执行的时候 按M可以切换为按照内存进行排序.

                   3. ~~~shell
                      [root@bogon bin] echo $$   打印当前bash的pid
                      128757
                      [root@bogon bin] top -d 2 -p 128757  ## 只观察当前bash进程
                      top - 17:56:13 up 3 days, 17:23,  2 users,  load average: 0.01, 0.02, 0.05
                      Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
                      %Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
                      KiB Mem :   995684 total,    70884 free,   313624 used,   611176 buff/cache
                      KiB Swap:   524284 total,   401148 free,   123136 used.   489020 avail Mem 
                      
                         PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                       
                      128757 root      20   0  115880   2360   1792 S  0.0  0.2   0:00.04 bash  
                      ~~~

               11. nice [-n 数字] command 给 命令赋值一个优先值  root用户可以调整的范围是:-20-19 一般用户调整的范围是:0-19.一般用户只能越调越高.

               12. renice [number] PID 已存在的进程的nice重新调整.

               13. 系统资源的观察

                   1. free 观察内存使用情况.
                   
                   2. uname:查阅系统和核心相关信息.
                   
                   3. netstat:追踪网络或socket文件
                   
                      1. ~~~shell
                         [root@localhost /]# netstat
                         Active Internet connections (w/o servers)
                         Proto Recv-Q Send-Q Local Address           Foreign Address         State
                         tcp        0      0 localhost.localdoma:ssh 192.168.199.1:58479     ESTABLISHED
                         tcp        0      0 localhost.localdoma:ssh 192.168.199.1:59255     ESTABLISHED
                         tcp        0      0 localhost.localdoma:ssh 192.168.199.1:58480     ESTABLISHED
                         tcp        0     48 localhost.localdoma:ssh 192.168.199.1:59254     ESTABLISHED
                         Active UNIX domain sockets (w/o servers)
                         Proto RefCnt Flags       Type       State         I-Node   Path
                         unix  2      [ ]         DGRAM                    11718    /run/systemd/shutdownd
                         unix  3      [ ]         DGRAM                    9177     /run/systemd/notify
                         unix  2      [ ]         DGRAM                    9179     /run/systemd/cgroups-agent
                         #两部分组成:网络的连接和Linux上面的socket进程相关性部分.
                         ## Recv-Q 非由用户进程连接到此socket的复制的总Bytes数
                         ## Local Addrs 本地端的 ip port 情况.
                         ## State 连接状态 :建立(ESTABLISHED) 和监听(LISTEN)
                         # 第二部分socket文件:可以沟通两个进程之间的信息.
                         ## RefCnt:连接到此socket的进程的数量.
                         ## Type:socket存取的类型,主要有确认连接的STERAM和不需要确认的DGRAM两种.
                         ## State:如果为CONNECTED表示多个进程之间已经建立了连接.
                         ## Path:连接到此socket的相关进程的路径,或是相关数据输出的路径.
                         ~~~
                   
                      2. ~~~shell
                         [root@localhost /] netstat -tulnp # 找出目前系统上已在监听的网络连接以及pid
                         Active Internet connections (only servers)
                         Proto Recv-Q Send-Q  Local Address           Foreign Address         State
                         tcp        0      0  0.0.0.0:2181            0.0.0.0:*               LISTEN    PID/Program name
                         101762/docker-proxy
                         ~~~
                   
                      3. ~~~shell
                         [root@localhost /]# pidof systemd rsyslogd
                         1 36720
                         ## 根据服务名字找到pid
                         ~~~
                   
                   4. SELiuux 部分以后再补充

          21. 认识系统服务

              1. 服务(deamon):一直在内存中的进程,就是服务.deamon是service的背景运行程序.

              2. ~~~shell
                 [root@bogon /] systemctl  #  列出系统上面所有启动的unit(单一服务)
                   UNIT                                         LOAD   ACTIVE SUB       DESCRIPTION
                   proc-sys-fs-binfmt_misc.automount            loaded active running   Arbitrary Executable File Formats File Syste  sys-devices-pci0000:00-0000:00:07.1-ata2-host2-target2:0:0-2:0:0:0-block-sr0.device loaded active plugged   VMwar  sys-devices-pci0000:00-0000:00:10.0-host0-target0:0:0-0:0:0:0-block-sda-sda1.device loaded active plugged   VMwar
                   # UNIT:项目名称 
                   # LOAD:开机时是否被加载
                   # ACTIVE:状态 与SUB搭配,就是使用systemctl status查询到的
                   # DESCRIPTION:描述
                 ~~~

              3. ~~~shell
                 [root@bogon /] systemctl list-unit-files ## 查询所有已经安装的unit有哪些
                 UNIT FILE                                     STATE   
                 proc-sys-fs-binfmt_misc.automount             static  
                 dev-hugepages.mount                           static  
                 dev-mqueue.mount                              static  
                 ~~~

              4. ~~~shell
                 [root@bogon /] systemctl list-units --type=service | grep ssh  ## 查询系统是否有名称为ssh的服务
                   sshd.service                       loaded active running OpenSSH server daemon
                 ~~~

              5. ~~~shell
                 [root@bogon /] systemctl list-dependencies  ### 展示所有服务的依赖关系
                 ● ├─basic.target
                 ● │ ├─microcode.service
                 ● │ ├─rhel-dmesg.service
                 ● │ ├─selinux-policy-migrate-local-changes@targeted.service
                 ● │ ├─paths.target
                 ● │ ├─slices.target
                 ● │ │ ├─-.slice
                 ● │ │ └─system.slice
                 ● │ ├─sockets.target
                 ● │ │ ├─dbus.socket
                 ~~~

              6. ~~~shell
                 [root@bogon /] systemctl list-sockets ##列出所有的socket文件
                 LISTEN                      UNIT                         ACTIVATES
                 /dev/log                    systemd-journald.socket      systemd-journald.service
                 /run/dbus/system_bus_socket dbus.socket                  dbus.service
                 ~~~

              7. 服务文件的内容,写一个自己的服务.


## 实战

### 检查端口占用

~~~shell
## 以2181 端口为例
sudo lsof -i:2181
COMMAND   PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
java    12533 root  118u  IPv6 50365908      0t0  TCP *:eforward (LISTEN)
java    12533 root  122u  IPv6 50366228      0t0  TCP localhost:eforward->localhost:34060 (ESTABLISHED)
java    13232 root  118u  IPv6 50367691      0t0  TCP localhost:34060->localhost:eforward (ESTABLISHED)
## 查看线程PID为12533 
ps -ef | grep 12533
root     12533     1  0 13:42 pts/1    00:00:01 /baicmotor_vsm/jdk/bin/java -Xmx512M -Xms512M -server -XX:+UseG1GC -XX:MaxGCPauseMillis=2..
## 杀死线程
kill -9 12533

~~~























