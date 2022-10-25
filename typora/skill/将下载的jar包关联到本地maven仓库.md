#### 现到maven repository 中下载jar包到本地

#### 然后执行一下命令

~~~bash
mvn install:install-file -Dfile="D:\httpmime-4.5.9.jar" -DgroupId=org.apache.httpcomponents -DartifactId=httpmime -Dversion=4.5.9 -Dpackaging=jar
#### 参数说明
## Dfile:本地下载jar包的路径
~~~

#### 在pom.xml引入对应的版本即可.