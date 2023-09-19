1. 在tomcat/bin下的catalina.sh上边添加下边的一段设置

```sh
CATALINA_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,address=60222,suspend=n,server=y"
```