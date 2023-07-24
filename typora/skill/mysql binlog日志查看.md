# 操作方法

## 命令

~~~sql
show binary logs;  ## 查询有那些binlog日志
show master status; ## 正在跟新的binlog
show binlog events in 'binlog.000013' from 689129   limit 20; ## 开始位置为689129 bin日志
~~~

## 脚本查询

~~~sh
## 显示不友好 用mysql的start与插件的stop-position结合的使用.-vv才能显示具体的操作语句. base64-output 格式化显示
./mysqlbinlog -vv --start-datetime='2023-07-17 15:57:00' --stop-datetime='2023-07-17 15:59:00' --stop-position=692788 -vv  --base64-output=decode-rows  --database=nacos  /gac_vsm/serve/mysql/data/binlog.000013
~~~



