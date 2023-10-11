~~~shell
## 查看占用端口号进程
netstat -aon|findstr "端口号"
C:\Users\b9082>netstat -ano | findstr 8080
  TCP    10.63.74.205:7414      42.81.237.222:8080     CLOSE_WAIT      66716
  TCP    10.63.74.205:7417      42.81.237.222:8080     CLOSE_WAIT      66716
  TCP    10.63.74.205:7466      101.91.33.235:8080     ESTABLISHED     66716
## 查看进程详细信息
tasklist|findstr "进程号"
C:\Users\b9082>tasklist|findstr "66716"
WXWork.exe                   66716 Console                   45    225,968 K
## 杀死任务
taskkill /T /F /PID 进程号
~~~

