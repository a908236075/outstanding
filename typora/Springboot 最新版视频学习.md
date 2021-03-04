## Springboot2

#### 配置

1. ```java
   // 改变默认当前包及其子包扫描路径
   @SpringBootApplication(scanBasePackages = "com.xiaobo")
   ```

2. 各种配置都有默认值,spring-boot-autoconfigure

3. 自定义配置

   1. @Configuration  @Bean  默认为单例

   2. proxyBeanMethods  代理Bean的方法. 检查组件在容器中是否拥有.也就是说,如果不想此配置只有一个单例,例如有其它成员可能会new我的类,就设置成false

   3. ~~~java
      @Configuration(proxyBeanMethods = true)
      ~~~

4. @Import 可以导入配置

5. @Conditional  条件配置  满足条件后 此类定义的配置才能够生效.

6. @ImportSource 导入资源 例如自定义一个xml,默认并没有加载,用此注解导入.

7. @ConfigurationProperties(prefix ="mycar") 配置文件中编辑了mycar相关的属性,就能自动将配置文件中的属性读取,并封装在类上面.此注解要结合@component注解一起使用.或者在**配置类**上使用@EnableAutoConfiguration(Car.class)两种等价的方法,方便引入第三方的包.

   1. ### @Component + @ConfigurationProperties

   2.   **@EnableConfigurationProperties + @ConfigurationProperties**

8. ```java
   @SpringBootApplication
   // 等价于
   @SpringBootConfiguration
   @EnableAutoConfiguration   // 自动导入配置类  按需导入
   @ComponentScan
   ```

9. Spring会默认配置很多配置类,如果用户自己配置了就使用用户自己的,例如注解 @ConditionalOnMissingBean.每个配置有默认的配置文件 XXXProperties(是以类的形式存储的配置信息)
   XXXAutoConfiguration -------------->组件-------------->XXXProperties------->application.properties
   所以我们最后在application.properties中定义的属性值是生效的.

10. 在application的配置文件中配置debug=true 开启自动配置报告  进行配置分析

### Web

1. 只要静态资源放在类路径下： called `/static` (or `/public` or `/resources` or `/META-INF/resources`

   访问 ： 当前项目根路径/ + 静态资源名

2. ~~~yaml
   spring:
     mvc:
       static-path-pattern: /res/**
     resources:
       static-locations: [classpath:/haha/]
   ~~~

3. 配置欢迎页和自定义Favicon

   1. ~~~yaml
      spring:
      #  mvc:
      #    static-path-pattern: /res/**   这个会导致 欢迎页 和 Favicon 功能失效
      ~~~

4. 