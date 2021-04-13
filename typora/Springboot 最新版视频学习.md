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

3. #### 配置欢迎页和自定义Favicon

   1. ~~~yaml
      spring:
      #  mvc:
      #    static-path-pattern: /res/**   这个会导致 欢迎页 和 Favicon 功能失效
      ~~~

4. 默认资源处理的代码

   1. Springboot启动的时候回默认加载WebMvcAutoConfiguration这个是和SpringMVC相关的配置类。
      
   2. 一般碰见**EnableConfigurationProperties**时候是可以配置文件进行绑定，也就在application.yaml中就可以是配置的属性生效.例如**WebMvcProperties**,是以spring.mvc开头的配置属性.

   3. 只有一个有参构造器

      1. ~~~java
           //有参构造器所有参数的值都会从容器中确定
         
         //ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
         
         //WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
         
         //ListableBeanFactory beanFactory Spring的beanFactory
         
         //HttpMessageConverters 找到所有的HttpMessageConverters
         
         //ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
         
         //DispatcherServletPath  
         
         //ServletRegistrationBean   给应用注册Servlet、Filter....
         
             public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
         
                         ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
         
                         ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
         
                         ObjectProvider<DispatcherServletPath> dispatcherServletPath,
         
                         ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
         
                     this.resourceProperties = resourceProperties;
         
                     this.mvcProperties = mvcProperties;
         
                     this.beanFactory = beanFactory;
         
                     this.messageConvertersProvider = messageConvertersProvider;
         
                     this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
         
                     this.dispatcherServletPath = dispatcherServletPath;
         
                     this.servletRegistrations = servletRegistrations;
         
                 }
         ~~~

      2. ##### 资源处理的默认规则

      ~~~java
      protected void addResourceHandlers(ResourceHandlerRegistry registry) {
                  super.addResourceHandlers(registry);
          		// add-mapping false  可以禁用静态资源的访问
                  if (!this.resourceProperties.isAddMappings()) {
                      logger.debug("Default resource handling disabled");
                  } else {
                      ServletContext servletContext = this.getServletContext();
                      this.addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
                      // 如果访问的路径是/** 则去加载this.resourceProperties.getStaticLocations()这个路径,为默认配置四个路径: "classpath:/META-INF/resources/",
      				"classpath:/resources/", "classpath:/static/", "classpath:/public/"
                      this.addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
                          registration.addResourceLocations(this.resourceProperties.getStaticLocations());
                          if (servletContext != null) {
                              registration.addResourceLocations(new Resource[]{new ServletContextResource(servletContext, "/")});
                          }
      
                      });
                  }
              }
      ~~~

   4. ##### 配置欢迎页

      1. ~~~java
         //  HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。  
         @Bean
         public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
         				FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
             // 欢迎页的配置路径this.mvcProperties.getStaticPathPattern() 必须是/**
         			WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
         					new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
         					this.mvcProperties.getStaticPathPattern());
         			welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
         			welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
         			return welcomePageHandlerMapping;
         		}
         ~~~

   5. #### rest分隔需要在springboot中配置开启:

      1. SpringMVC默认开启了方法过滤

         1. ~~~java
            // 当没有HiddenHttpMethodFilter时候生效
            // spring.mvc.hiddenmethod.filter 为enabled时候生效 需要配置在文件中配置
            // matchIfMissing=false 表示配置了就生效不会忽略
            @Bean
            @ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
            @ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
            	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
            		return new OrderedHiddenHttpMethodFilter();
            	}
            ~~~

         2. ##### 方法断点进入HiddenHttpMethodFilter的doFilterInternal方法中

            1. ~~~java
               @Override
               protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
               			throws ServletException, IOException {
               
               		HttpServletRequest requestToUse = request;
               		// 必须为post请求才会处理
               		if ("POST".equals(request.getMethod()) && request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) == null) {
                           // this.methodParam 即使封装的属性 _method
               			String paramValue = request.getParameter(this.methodParam);
               			if (StringUtils.hasLength(paramValue)) {
               				String method = paramValue.toUpperCase(Locale.ENGLISH);
                // 判断是否为允许的请求   Collections.unmodifiableList(Arrays.asList(HttpMethod.PUT.name(),
               //					HttpMethod.DELETE.name(), HttpMethod.PATCH.name())); 
                               // patch做局部更新的请求
               				if (ALLOWED_METHODS.contains(method)) {
               					requestToUse = new HttpMethodRequestWrapper(request, method);
               				}
               			}
               		}
               
               		filterChain.doFilter(requestToUse, response);
               	}
               ~~~

      2. ~~~yaml
         spring:
           # rest 风格请求的编写
           mvc:
             hiddenmethod:
               filter:
                 enabled: true
         ~~~

      3. ##### 当put请求和delete请求 需要不想用_method封装的时候

         1. ~~~java
            //自定义filter 自定义变量名称为 _m
            // 例如想要发送delete或者put请求,需要发送post请求 然后携带一个_method属性名为delete
                @Bean
                public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
                    HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
                    methodFilter.setMethodParam("_m");
                    return methodFilter;
                }
            ~~~

   6. 请求映射原理

      1. ![](.\picture\springboot 最新版学习\SpringMVC发送http请求底层执行的方法.png)
      2. SpringMVC所有请求的开始**DispatchServlet**,HttpServlet是他的父类的父类.
      3. 映射路径 :HttpServlet(doGet)  找子类-->HttpServletBean(doGet) -->FrameworkServlet(doGet) 调用了processRequest--->里面调用了doService找它在子类DispatcherServlet的实现调用了-----> doDispatch 分析  mappedHandler = this.getHandler(processedRequest); 方法.
         AbstractHandlerMethodMapping  中的  getHandlerInternal  
      4. 
      5. 可以学习一下ReentrantReadWriteLock

   7. 普通参数与基本注解

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

   8. 参数处理

      1. HandlerMapping中找到能处理请求的Handler（Controller.method()）
      2. 为当前Handler 找一个适配器 HandlerAdapter； **RequestMappingHandlerAdapter**
      3. 适配器执行目标方法并确定方法参数的每一个值.

   9. 找到Handler 找到adapter 就去执行handle方法.

      ```java
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
      ```

   10. handle方法里面,遇到了参数argumentResolvers,里面保存了所有的**参数解析器**.同理有方法的返回值处理器.

   11. ```java
       mav = invokeHandlerMethod(request, response, handlerMethod); //执行目标方法
       //ServletInvocableHandlerMethod
       Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
       //获取方法的参数值 
       Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
       ```

   12. Map<String,Object> map,  Model model, HttpServletRequest request 都是可以给request域中放数据，request.getAttribute();

   13. 处理参数

      14. **WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);**

      **WebDataBinder :web数据绑定器，将请求参数的值绑定到指定的JavaBean里面**

      **WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中**

      

      ​	2. **GenericConversionService：在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（JavaBean -- Integer）**

      **byte -- > file**

4. #### 请求映射原理

   1. 请求由HttpServletBean的doGet方法处理,Httpservlet处理是所有请求的父类,最后走到实现方法DispatchServlet的doService的**dodispatch方法**中.

        ![request请求](.\picture\springboot 最新版学习\request请求.png)

   2. doDispatch方法中会**寻找handle**r,也就是请求路径对应的controller.method,所有的handler多会被注册在registerHandler中,最终进行循环遍历匹配.默认的为requestMappingHandlerMapping.
   
   3. 为当前handler找到一个**适配器**,handlerAdapter,默认为RequestMappingHandlerMappingAdapter,适配器执行目标方法,确定参数的每一个值.
   
   4. **参数解析器-HandlerMethodArgumentResolver**,SpringMVC目标方法能写多少种参数,取决于有多少种参数解析器.例如 **ServletRequestMethodArgumentResolver** 处理HttpServletRequest 请求.
      
      1. **复杂参数:** **Map、Model类型的参数**，会返回 mavContainer.getModel（）；---> BindingAwareModelMap 是Model 也是Map 
      
   5. **返回值处理器**
   
        1. modleAndViewContain将数据取出来,从新封装了modleAndView中包含了参数和视图,
   
        2. ```xml
             调用各自 HandlerMethodArgumentResolver 的 resolveArgument 方法即可
             ```
   
   6. 处理派发结果
   
      1.  **processDispatchResult**(processedRequest, response, mappedHandler, mv, dispatchException); 
   
      2.  renderMergedOutputModel(mergedModel, getRequestToExpose(request), response); 
   
      3. ~~~java
         InternalResourceView：
         @Override
         	protected void renderMergedOutputModel(
         			Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
         
         		// Expose the model object as request attributes.
         		exposeModelAsRequestAttributes(model, request);
         
         		// Expose helpers as request attributes, if any.
         		exposeHelpers(request);
         
         		// Determine the path for the request dispatcher.
         		String dispatcherPath = prepareForRendering(request, response);
         
         		// Obtain a RequestDispatcher for the target resource (typically a JSP).
         		RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
         		if (rd == null) {
         			throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
         					"]: Check that the corresponding file exists within your web application archive!");
         		}
         
         		// If already included or response already committed, perform include, else forward.
         		if (useInclude(request, response)) {
         			response.setContentType(getContentType());
         			if (logger.isDebugEnabled()) {
         				logger.debug("Including [" + getUrl() + "]");
         			}
         			rd.include(request, response);
         		}
         
         		else {
         			// Note: The forwarded resource is supposed to determine the content type itself.
         			if (logger.isDebugEnabled()) {
         				logger.debug("Forwarding to [" + getUrl() + "]");
         			}
         			rd.forward(request, response);
         		}
         	}
         ~~~
   
      4. ~~~java
         // 暴露模型作为请求域属性
         // Expose the model object as request attributes.
         exposeModelAsRequestAttributes(model, request);
         ~~~
   
      5. ~~~java
         protected void exposeModelAsRequestAttributes(Map<String, Object> model,
         			HttpServletRequest request) throws Exception {
         
             //model中的所有数据遍历挨个放在请求域中
         		model.forEach((name, value) -> {
         			if (value != null) {
         				request.setAttribute(name, value);
         			}
         			else {
         				request.removeAttribute(name);
         			}
         		});
         	}
         ~~~
   
   7. POJO封装过程
   
      1.  **ServletModelAttributeMethodProcessor** 这个参数解析器进行解析
   
      2. **WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);**
   
      3. **WebDataBinder :web数据绑定器，将请求参数的值绑定到指定的JavaBean里面**.
   
      4. **WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中**
   
      5. **GenericConversionService：在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（JavaBean -- Integer）**
   
         **byte -- > file**
   
      6. 自定义conversion
   
         1. 未来我们可以给WebDataBinder里面放自己的Converter；
   
         2. **private static final class** StringToNumber<T **extends** Number> **implements** Converter<String, T>
   
         3. 重写WebMvcConfigurer中addFormatter方法.
   
         4. ~~~java
                //1、WebMvcConfigurer定制化SpringMVC的功能
                @Bean
                public WebMvcConfigurer webMvcConfigurer(){
                    return new WebMvcConfigurer() {
                        @Override
                        public void configurePathMatch(PathMatchConfigurer configurer) {
                            UrlPathHelper urlPathHelper = new UrlPathHelper();
                            // 不移除；后面的内容。矩阵变量功能就可以生效
                            urlPathHelper.setRemoveSemicolonContent(false);
                            configurer.setUrlPathHelper(urlPathHelper);
                        }
            
                        @Override
                        public void addFormatters(FormatterRegistry registry) {
                            registry.addConverter(new Converter<String, Pet>() {
            
                                @Override
                                public Pet convert(String source) {
                                    // 啊猫,3
                                    if(!StringUtils.isEmpty(source)){
                                        Pet pet = new Pet();
                                        String[] split = source.split(",");
                                        pet.setName(split[0]);
                                        pet.setAge(Integer.parseInt(split[1]));
                                        return pet;
                                    }
                                    return null;
                                }
                            });
                        }
                    };
                }
            ~~~
   
5. #### 数据响应与内容协商

   1. ~~~java
      try {		// 处理返回值的方法
      			this.returnValueHandlers.handleReturnValue(
      					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
      		}
      ~~~

   2. 遍历**返回值处理器**判断是否支持这种类型返回值 supportsReturnType

   3. 返回值处理器调用 handleReturnValue 进行处理

   4. RequestResponseBodyMethodProcessor 可以处理返回值标了@ResponseBody 注解的。

      1. 内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）
      2. 服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据，
      3. SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？
         - 得到MappingJackson2HttpMessageConverter可以将对象写为json
         - 利用MappingJackson2HttpMessageConverter将对象转为json再写出去。

6. #### 内容协商

   1. 内容协商原理

      1. 判断当前响应头中是否已经有确定的媒体类型。MediaType
      2. **获取客户端（PostMan、浏览器）支持接收的内容类型。（获取客户端Accept请求头字段）【application/xml】**
         - **contentNegotiationManager 内容协商管理器 默认使用基于请求头的策略**
      3.  遍历循环所有当前系统的 **MessageConverter**，看谁支持操作这个对象（Person） 
      4.  找到支持操作Person的converter，把converter支持的媒体类型统计出来。 
      5.  客户端需要【application/xml】。服务端能力【10种、json、xml】 
      6.  进行内容协商的最佳匹配媒体类型 
      7.  用 支持 将对象转为 最佳匹配媒体类型 的converter。调用它进行转化 。 

   2. 默认的**媒体协商策略**是只需要改变请求头中Accept字段。Http协议中规定的，告诉服务器本客户端可以接收的数据类型。 我们可以使用**参数方式内容协商策略**功能.

      1. ~~~yaml
         spring:
             contentnegotiation:
             favor-parameter: true  #开启请求参数内容协商模式
         ~~~

      2. 发送请求   http://localhost:8080/test/person?format=json 或 [http://localhost:8080/test/person?format=](http://localhost:8080/test/person?format=json)xml

      3. **参数策略优先于媒体协商策略**,最终将我们定义的格式返回.

