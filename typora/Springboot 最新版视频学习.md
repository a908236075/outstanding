## Springboot2

#### 代码和文档

​	本视频文档地址：https://yuque.com/atguigu/springboot

​	本视频源码地址：https://gitee.com/leifengyang/springboot2

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

#### yaml

1. 属性封装

   1. ~~~java
      @Data
      public class Person {
      	
      	private String userName;
      	private Boolean boss;
      	private Date birth;
      	private Integer age;
      	private Pet pet;
      	private String[] interests;
      	private List<String> animal;
      	private Map<String, Object> score;
      	private Set<Double> salarys;
      	private Map<String, List<Pet>> allPets;
      }
      
      @Data
      public class Pet {
      	private String name;
      	private Double weight;
      }
      ~~~

   2. ~~~yaml
      # yaml表示以上对象
      person:
        userName: zhangsan
        boss: false
        birth: 2019/12/12 20:12:33
        age: 18
        pet: 
          name: tomcat
          weight: 23.4
        interests: [篮球,游泳]
        animal: 
          - jerry
          - mario
        score:
          english: 
            first: 30
            second: 40
            third: 50
          math: [131,140,148]
          chinese: {first: 128,second: 136}
        salarys: [3999,4999.98,5999.99]
        allPets:
          sick:
            - {name: tom}
            - {name: jerry,weight: 47}
          health: [{name: mario,weight: 47}]
      ~~~

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

4. 默认资源处理的代码

   1. ~~~java
      protected void addResourceHandlers(ResourceHandlerRegistry registry) {
                  super.addResourceHandlers(registry);
                  if (!this.resourceProperties.isAddMappings()) {
                      logger.debug("Default resource handling disabled");
                  } else {
                      ServletContext servletContext = this.getServletContext();
                      this.addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
                      this.addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
                          registration.addResourceLocations(this.resourceProperties.getStaticLocations());
                          if (servletContext != null) {
                              registration.addResourceLocations(new Resource[]{new ServletContextResource(servletContext, "/")});
                          }
      
                      });
                  }
              }
      ~~~

   2. rest分隔需要在springboot中配置开启:

      1. ~~~yaml
         spring:
           # rest 风格请求的编写
           mvc:
             hiddenmethod:
               filter:
                 enabled: true
         ~~~

      2. 当put请求和delete请求 需要不想用_method封装的时候

         1. ~~~java
            //自定义filter 自定义变量名称为 _m
                @Bean
                public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
                    HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
                    methodFilter.setMethodParam("_m");
                    return methodFilter;
                }
            ~~~

   3. 请求映射原理

      1. ![image-20210304131416974](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20210304131416974.png)
      2. 映射路径 :HttpServlet(doGet)  找子类-->HttpServletBean(doGet) -->FrameworkServlet(doGet) 调用了processRequest--->里面调用了doService找它在子类DispatcherServlet的实现调用了-----> doDispatch 分析  mappedHandler = this.getHandler(processedRequest); 方法.
         AbstractHandlerMethodMapping  中的  getHandlerInternal  
      3. 可以学习一下ReentrantReadWriteLock

   4. 普通参数与基本注解

      1. @PathVariable、@RequestHeader、@ModelAttribute、@RequestParam、@RequestBody   @MatrixVariable、@CookieValue

      2. ~~~java
         @RestController
         public class ParameterTestController {
             //  car/2/owner/zhangsan
             @GetMapping("/car/{id}/owner/{username}")
             public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                              @PathVariable("username") String name,
                                              @PathVariable Map<String,String> pv,
                                              @RequestHeader("User-Agent") String userAgent,
                                              @RequestHeader Map<String,String> header,
                                              @RequestParam("age") Integer age,
                                              @RequestParam("inters") List<String> inters,
                                              @RequestParam Map<String,String> params,
                                              @CookieValue("_ga") String _ga,
                                              @CookieValue("_ga") Cookie cookie){
         
                 Map<String,Object> map = new HashMap<>();
         //        map.put("id",id);
         //        map.put("name",name);
         //        map.put("pv",pv);
         //        map.put("userAgent",userAgent);
         //        map.put("headers",header);
                 map.put("age",age);
                 map.put("inters",inters);
                 map.put("params",params);
                 map.put("_ga",_ga);
                 System.out.println(cookie.getName()+"===>"+cookie.getValue());
                 return map;
             }
         
         
             @PostMapping("/save")
             public Map postMethod(@RequestBody String content){
                 Map<String,Object> map = new HashMap<>();
                 map.put("content",content);
                 return map;
             }
         
             //1、语法： 请求路径：/cars/sell;low=34;brand=byd,audi,yd
             //2、SpringBoot默认是禁用了矩阵变量的功能
             //      手动开启：原理。对于路径的处理。UrlPathHelper进行解析。
             //              removeSemicolonContent（移除分号内容）支持矩阵变量的
             //3、矩阵变量必须有url路径变量才能被解析   {path}
             @GetMapping("/cars/{path}")
             public Map carsSell(@MatrixVariable("low") Integer low,
                                 @MatrixVariable("brand") List<String> brand,
                                 @PathVariable("path") String path){
                 Map<String,Object> map = new HashMap<>();
         
                 map.put("low",low);
                 map.put("brand",brand);
                 map.put("path",path);
                 return map;
             }
             // /boss/1;age=20/2;age=10
         
             @GetMapping("/boss/{bossId}/{empId}")
             public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                             @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
                 Map<String,Object> map = new HashMap<>();
         
                 map.put("bossAge",bossAge);
                 map.put("empAge",empAge);
                 return map;
             }
         }
         ~~~

   5. 参数处理

      1. HandlerMapping中找到能处理请求的Handler（Controller.method()）
      2. 为当前Handler 找一个适配器 HandlerAdapter； **RequestMappingHandlerAdapter**
      3. 适配器执行目标方法并确定方法参数的每一个值.

   6. 找到Handler 找到adapter 就去执行handle方法.

      ```java
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
      ```

   7. handle方法里面,遇到了参数argumentResolvers,里面保存了所有的**参数解析器**.同理有方法的返回值处理器.

   8. ```java
      mav = invokeHandlerMethod(request, response, handlerMethod); //执行目标方法
      //ServletInvocableHandlerMethod
      Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
      //获取方法的参数值 
      Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
      ```

   9. Map<String,Object> map,  Model model, HttpServletRequest request 都是可以给request域中放数据，request.getAttribute();

   10. 处理参数

      1. **WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);**

      **WebDataBinder :web数据绑定器，将请求参数的值绑定到指定的JavaBean里面**

      **WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中**

      

      ​	2. **GenericConversionService：在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（JavaBean -- Integer）**

      **byte -- > file**

   12. 

