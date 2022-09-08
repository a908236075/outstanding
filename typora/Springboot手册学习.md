# Spring Boot Tutorials

1. 在pom.xml中可以定义启动类和跳过test

   - ~~~xml
      <artifactId>outstanding</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <packaging>war</packaging> // 打war包
     <properties>
             <java.version>1.8</java.version>
            <start-class>com.liuliu.outstanding.controller.SayHello</start-class>
            <maven.test.skip>true</maven.test.skip>
     </properties>
     ~~~

2. 