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

3. #### 配置欢迎页和自定义Favicon

   1. ~~~yaml
      spring:
      #  mvc:
      #    static-path-pattern: /res/**   这个会导致 欢迎页 和 Favicon 功能失效
      ~~~

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
   
5. #### 数据相应于内容协商