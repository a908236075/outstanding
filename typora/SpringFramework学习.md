#  1.IOC容器

## Bean的定义

### 什么是Bean

​	Bean是程序的骨架,Spring IOC 管理着这些Bean,Bean的层级关系反映到容器使用的配置元数据中.

### Spring IOC 的基础

​	org.springframework.beans 和 org.springframework.context 是 Spring IOC 的基础.

### 定义Bean的形式

1. 需要通过注解或者xml的形式定义Bean,定义的类与容器真正使用的类相对应.类可以相互依赖.

2. ~~~xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   	<!-- 对于复杂的定义可以使用import --> 
       <!-- service.xml必须是在同一个文件下-->
        <import resource="services.xml"/>
       <import resource="resources/messageSource.xml"/>
       <import resource="/resources/themeSource.xml"/>
   
       <bean id="..." class="...">  
           <!-- collaborators and configuration for this bean go here -->
       </bean>
   
       <bean id="..." class="...">
           <!-- collaborators and configuration for this bean go here -->
       </bean>
        <!-- services 类可以相互依赖 -->
       <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
           <property name="accountDao" ref="accountDao"/>
           <property name="itemDao" ref="itemDao"/>
           <!-- additional collaborators and configuration for this bean go here -->
       </bean>
       
       <bean id="accountDao"
           class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
           <!-- additional collaborators and configuration for this bean go here -->
       </bean>
       
       <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
           <!-- additional collaborators and configuration for this bean go here -->
       </bean>
       
   
       <!-- more bean definitions go here -->
   
   </beans>
   ~~~

### ClassPathXmlApplicationContext

1. 定义后可以通过ClassPathXmlApplicationContext获取到

2. ~~~java
   // create and configure beans
   ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
   
   // retrieve configured instance
   PetStoreService service = context.getBean("petStore", PetStoreService.class);
   
   // use configured instance
   List<String> userList = service.getUsernameList();
   ~~~

### FactoryBean与BeanFactory

​	在Spring文档中，“factory bean”指的是在Spring容器中配置的bean，它通过实例或静态工厂方法创建对象。相比之下，FactoryBean(注意大写)指的是spring特定的FactoryBean实现类。

## 控制反转

### 定义:

​	控制反转又名依赖注入,原本Bean的实例化需要Bean层层的构建依赖的Bean,现在由容器构建完成,可直接使用Bean."获得依赖对象的过程被反转了",控制被反转之后，获得依赖对象的过程由自身管理变为了由IOC容器主动注入.所谓依赖注入，就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。

### 依赖注入实现

#### 构造方法的注入(静态工厂方法和它类似)

~~~xml
<beans>
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
</beans>

<!--属性注入-->
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
~~~



~~~java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
~~~



#### Set方法注入

~~~xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>S
~~~

### 构造方法还是set方法注入如何选?

1. 构造方法是在创建类的时候就调用,所以一般会将必选的依赖类通过构造方法注入,可选的通过set方法来注入.
2. set方式注入可以解决依赖循环的问题.

### 参数赋值的时机

1. Spring在构造类的时候只会对类型进行校验,而参数的值是在真正调用类的时候才会赋值.**????**

2. ~~~java
   @AllArgsConstructor
   @NoArgsConstructor
   @Mapper
   public class BeanProcessMapper {
       private String isBetter = "zhagnsan";
   
       public String getIsBetter() {
           return isBetter;
       }
   }
   
   AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(BeanProcessMapper.class);
           ConfigurableListableBeanFactory beanFactory = annotationConfigApplicationContext.getBeanFactory();
           BeanProcessMapper beanProcessMapper = annotationConfigApplicationContext.getBean("beanProcessMapper", BeanProcessMapper.class);
           System.out.println(beanProcessMapper.getIsBetter());
   ~~~

3. 实际情况并非如此 ,在我未调用时候,发现在BeanFactory中 BeanProcessMapper的name值已经静静的躺在那里了.

   ![](.\picture\springFramework\类参数赋值.png)

## 属性绑定

1. ~~~xml
   <bean id="theTargetBean" class="..."/>
   
   <bean id="theClientBean" class="...">
       <property name="targetName">
           <idref bean="theTargetBean"/>
       </property>
   </bean>
   <!--等价于-->
   <bean id="theTargetBean" class="..." />
   
   <bean id="client" class="...">
       <property name="targetName" value="theTargetBean"/>
   </bean>
   ~~~

2. ~~~xml
   
   <!--collection赋值-->
   <bean id="moreComplexObject" class="example.ComplexObject">
       <!-- results in a setAdminEmails(java.util.Properties) call -->
       <property name="adminEmails">
           <props>
               <prop key="administrator">administrator@example.org</prop>
               <prop key="support">support@example.org</prop>
               <prop key="development">development@example.org</prop>
           </props>
       </property>
       <!-- results in a setSomeList(java.util.List) call -->
       <property name="someList">
           <list>
               <value>a list element followed by a reference</value>
               <ref bean="myDataSource" />
           </list>
       </property>
       <!-- results in a setSomeMap(java.util.Map) call -->
       <property name="someMap">
           <map>
               <entry key="an entry" value="just some string"/>
               <entry key="a ref" value-ref="myDataSource"/>
           </map>
       </property>
       <!-- results in a setSomeSet(java.util.Set) call -->
       <property name="someSet">
           <set>
               <value>just some string</value>
               <ref bean="myDataSource" />
           </set>
       </property>
   </bean>
   
   <!--通过继承属性将Collection合并-->
   
   
   <beans>
       <bean id="parent" abstract="true" class="example.ComplexObject">
           <property name="adminEmails">
               <props>
                   <prop key="administrator">administrator@example.com</prop>
                   <prop key="support">support@example.com</prop>
               </props>
           </property>
       </bean>
       <bean id="child" parent="parent">
           <property name="adminEmails">
               <!-- the merge is specified on the child collection definition -->
               <props merge="true">
                   <prop key="sales">sales@example.com</prop>
                   <prop key="support">support@example.co.uk</prop>
               </props>
           </property>
       </bean>
   <beans>
   
   ~~~

3. ~~~xml
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   	<!--p-namespace可以简化属性赋值,他们是等价的,一般用于set方法-->
       <bean name="john-classic" class="com.example.Person">
           <property name="name" value="John Doe"/>
           <property name="spouse" ref="jane"/>
       </bean>
   
       <bean name="john-modern"
           class="com.example.Person"
           p:name="John Doe"
           p:spouse-ref="jane"/>
   
       <bean name="jane" class="com.example.Person">
           <property name="name" value="Jane Doe"/>
       </bean>
   </beans>
   
   
   <!--c-namespace简化构造方法的属性赋值-->
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="beanTwo" class="x.y.ThingTwo"/>
       <bean id="beanThree" class="x.y.ThingThree"/>
   
       <!-- traditional declaration with optional argument names -->
       <bean id="beanOne" class="x.y.ThingOne">
           <constructor-arg name="thingTwo" ref="beanTwo"/>
           <constructor-arg name="thingThree" ref="beanThree"/>
           <constructor-arg name="email" value="something@somewhere.com"/>
       </bean>
   
       <!-- c-namespace declaration with argument names -->
       <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
           c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>
   
   </beans>
   ~~~
   
4. ~~~xml
   <!--复合参数的赋值-->
   <bean id="something" class="things.ThingOne">
       <property name="fred.bob.sammy" value="123" />
   </bean>
   ~~~

5. 当依赖关系并不十分直接的时候可以用depend-on属性,多依赖.会在初始化beanOne之前初始化depends-on的类.

   ~~~xml
   
   
   <bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
       <property name="manager" ref="manager" />
   </bean>
   
   <bean id="manager" class="ManagerBean" />
   <bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
   
   ~~~

6. 定义一个懒加载的类,Application容器启动,不会创建懒加载的类,除非有非懒加载的类依赖它.

   ~~~xml
   <bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
   <bean name="not.lazy" class="com.something.AnotherBean"/>
   ~~~


   7. **单例依赖多例**:Bean的创建只在容器初始化的之后执行一次,当单例Bean依赖多例Bean的时候,要想每次拿到不同的Bean,可以实现ApplicationContextAware,调用getBean的方法,这样每次都从容器中取.

      1. ~~~java
         // a class that uses a stateful Command-style class to perform some processing
         package fiona.apple;
         
         // Spring-API imports
         import org.springframework.beans.BeansException;
         import org.springframework.context.ApplicationContext;
         import org.springframework.context.ApplicationContextAware;
         
         public class CommandManager implements ApplicationContextAware {
         
             private ApplicationContext applicationContext;
         
             public Object process(Map commandState) {
                 // grab a new instance of the appropriate Command
                 Command command = createCommand();
                 // set the state on the (hopefully brand new) Command instance
                 command.setState(commandState);
                 return command.execute();
             }
         
             protected Command createCommand() {
                 // notice the Spring API dependency!
                 return this.applicationContext.getBean("command", Command.class);
             }
         
             public void setApplicationContext(
                     ApplicationContext applicationContext) throws BeansException {
                 this.applicationContext = applicationContext;
             }
         }
         ~~~

      2. 注释事项:
         1. 动态代理使用cglib实现,所以用方法注入的类,不能用final修饰,子类也不行.
         2. 需要让容器扫描到这个类.
         3. 不能与@Bean这种注解混合使用,因为此类已经脱离了容器的管理.
         4. 方法注入还用在当单例对象依赖多例对象,每次想获取到不同的多例对象时候.
      3. @LookUp 检验使用的类是否和配置xml定义的类一致.

## Bean的生命周期

### 单例bean中有多例bean的依赖

- 实例化Bean只发生一次,所以依赖的多例Bean是同一个对象.因为多例的含义是，我们每次向Spring容器请求多例bean，都会创建一个新的对象返回。而B虽然是多例，但是我们是通过A访问B，并不是通过容器访问，所以拿到的永远是同一个B.

### 其它的生命周期

- 生命周期除了单例,多例之外,还有request,session,application,websocket,如果这些生命周期应用在web的请求中,需要spring配置Listener或者其他的监听器.

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

### 定义类的生命周期:

​	可以使用@RequestScope,@SessionScope,@ApplicationScope进行定义,也可以使用xml

1. ~~~java
   @RequestScope
   @Component
   public class AppPreferences {
       // ...
   }
   ~~~

2. ~~~xml
   <bean id="loginAction" class="com.something.LoginAction" scope="request"/>
   ~~~

### Scope类作为依赖:

1. @RequestScope,@SessionScope,@ApplicationScope这些web生命周期作为**依赖**的时候,需要生命Aop代理,这要每次依赖注入的类都是一个代理,如果没有生命,以为类的容器初始化仅在启动的时候执行,所以,每次取到的依赖都是相同的原始的定义的类.

2. ~~~xml
   ?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!-- an HTTP Session-scoped bean exposed as a proxy -->
       <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
           <!-- instructs the container to proxy the surrounding bean -->
           <aop:scoped-proxy/> 
       </bean>
   
       <!-- a singleton-scoped bean injected with a proxy to the above bean -->
       <bean id="userService" class="com.something.SimpleUserService">
           <!-- a reference to the proxied userPreferences bean -->
           <property name="userPreferences" ref="userPreferences"/>
       </bean>
   </beans>
   ~~~

## 定义Bean的性质

### 生命周期的回调

1. 通过实现**InitializingBean**接口,重写afterPropertiesSet方法以及实现**DisposableBean**接口 ,重写destroy方法 .

2. ~~~java
   public class AnotherExampleBean implements InitializingBean {
   
       @Override
       public void afterPropertiesSet() {
           // do some initialization work
       }
   }
   public class AnotherExampleBean implements DisposableBean {
   
       @Override
       public void destroy() {
           // do some destruction work (like releasing pooled connections)
       }
   }
   ~~~

## 类的继承关系

### xml定义类的继承关系

1. ~~~xml
   <bean id="inheritedTestBean" abstract="true"
           class="org.springframework.beans.TestBean">
       <property name="name" value="parent"/>
       <property name="age" value="1"/>
   </bean>
   
   <bean id="inheritsWithDifferentClass"
           class="org.springframework.beans.DerivedTestBean"
           parent="inheritedTestBean" init-method="initialize">  
       <property name="name" value="override"/>
       <!-- the age property value of 1 will be inherited from parent -->
   </bean>
   ~~~

2. 父类必须定义为**abstract=true**,抽象的父类与实例化的Bean不同,定义了Bean的公共属性,是子类的模板.

3. 如果不定义**abstract=true**,容器启动的时候会尝试实例化这个Bean.

## 容器扩展点

### `BeanPostProcessor`

1. BeanPostProcessor在此类中初始化类已经类的依赖管理,是类生成的容器,所以生成的过程都可以干预.
2. BeanPostProcessor的生命周期是容器(are scoped per-container),每个容器独有一个BeanPostProcessor,即使是由继承关系的容器创建的同一个类,他们依然是不同的.
3. `BeanFactoryPostProcessor` 是生产BeanPostProcessor的工厂.

## 基于注解的配置

### xml与注解比较

1. xml能够整体的定义类的各种依赖关系以及属性,可以在代码之外清晰的了解Bean的定义.
2. 注解配置更加的简单,写在代码中.减少了配置文件.

### 常用注解

#### 开启注解

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
~~~

1. 在Springboot项目默认已经开启了注解,无需单独配置
2. 开启注解后的加载顺序
   1. [`ConfigurationClassPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)
   2. [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)
   3. [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html)
   4. [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html)
   5. [`EventListenerMethodProcessor`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/context/event/EventListenerMethodProcessor.html)

#### @Required

~~~java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
~~~

1. @Required定义在当初始化类时**所依赖的类必须存在**,这要避免了依赖空指针异常.

#### @Autowired

1. 定义的位置: 构造方法,set方法或者依赖的类上.

   ~~~java
   public class MovieRecommender {
   
       private final CustomerPreferenceDao customerPreferenceDao;
       
       private MovieFinder movieFinder;
   
       @Autowired
       public void setMovieFinder(MovieFinder movieFinder) {
           this.movieFinder = movieFinder;
       }
   
   
       @Autowired
       private MovieCatalog movieCatalog;
   
       @Autowired
       public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
           this.customerPreferenceDao = customerPreferenceDao;
       }
   
       // ...
   }
   ~~~

2. @Autowired注解通过类型进行注入,当通过xml配置或者通过扫描注解配置的Bean,容器是知道对应的依赖关系的,但是对于 `@Bean` 工厂类的方法,直接使用会出现未有匹配的类问题.

   ~~~java
   @Configuration
   public class BeanConfig {
    
       @Bean
       public Person person() {
           return new Person("老王", 20);
       }
   }
   
   @Configuration
   public class ConfigBean {
   
       @Bean
       @Primary
       public BeanA firstBeanA() { return new BeanA(); }
   
       @Bean
       public BeanA secondBeanA() {  return new BeanA();}
   
   }
    
   ~~~

   ~~~xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           https://www.springframework.org/schema/context/spring-context.xsd">
   
       <context:annotation-config/>
   
       <bean class="example.SimpleMovieCatalog" primary="true">
           <!-- inject any dependencies required by this bean -->
       </bean>
   
       <bean class="example.SimpleMovieCatalog">
           <!-- inject any dependencies required by this bean -->
       </bean>
   
       <bean id="movieRecommender" class="example.MovieRecommender"/>
   
   </beans>
   ~~~

   解决方法:

   可用通过 `@Order`  或则`@Priority`注解.

   - @Priority与@Order类似，@Order是Spring提供的注解，@Priority是JSR 250标准，都是值越小优先级越高；
   - 与@Order不同，@Priority可以控制组件的**加载顺序**，因此@Priority侧重于单个注入的优先级排序；
   - @Priority优先级比@Order更高，两者共存时优先加载@Priority；
   - @Primary是优先级最高的，如果同时有@Primary、@Order、Ordered的话，@Primary注解的Bean会优先加；。
   - @Order实际作用在类型,影响类的加载,而不能定义类组件的顺序,即使它可以写在方法上.

3. 不是必须依赖类可以定义required=false,false属性的类注入的时候会被忽略.

   ~~~java
   public class SimpleMovieLister {
   
       private MovieFinder movieFinder;
   
       @Autowired(required = false)
       public void setMovieFinder(MovieFinder movieFinder) {
           this.movieFinder = movieFinder;
       }
   
       // ...
   }
   ~~~

   还可以通过其它的java8的Option,以及Spring Framework 5.0可以支持的@Nullable注解表示

   ```java
   public class SimpleMovieLister {
   
       @Autowired
       public void setMovieFinder(Optional<MovieFinder> movieFinder) {
           ...
       }
   }
   ```

   ```java
   public class SimpleMovieLister {
   
       @Autowired
       public void setMovieFinder(@Nullable MovieFinder movieFinder) {
           ...
       }
   }
   ```

4. 多个构造方法

   1. 单个参数的构造方法会默认被使用,如果有多个构造方法,且多个构造方法都使用了@Autowired,必须要request设置为false,以保证类使用此方法的可能,最终匹配参数最多的构造方法被使用,如果都**没有**使用@Autowired注解,如果存在默认或者主要的(primary/default constructor)将会被使用.

5. 注意:@Autowired`, `@Inject`, `@Value`, 和`@Resource这些注解都是通过BeanPostProcessor完成的,所以不能在BeanPostProcessor或者BeanFactoryPostProcessor使用这些注解,因为这些注解未完成时候根本不会被容器识别.

#### @Primary和@Qualifier

1. 当有同名同类型的类需要注入的时候,可以使用@Primary标记,一面单例的依赖出现混乱.例如多个厂商的数据库驱动都叫做driver一样.

2. ~~~java
   @Configuration
   public class MovieConfiguration {
   
       @Bean
       @Primary
       public MovieCatalog firstMovieCatalog() { ... }
   
       @Bean
       public MovieCatalog secondMovieCatalog() { ... }
   
       // ...
   }
   ~~~

3. @Qualifier

   1. ~~~java
      public class MovieRecommender {
      
          private MovieCatalog movieCatalog;
      
          private CustomerPreferenceDao customerPreferenceDao;
      
          @Autowired
          public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
                  CustomerPreferenceDao customerPreferenceDao) {
              this.movieCatalog = movieCatalog;
              this.customerPreferenceDao = customerPreferenceDao;
          }
      
          // ...
      }
      ~~~

   2. ~~~xml
      
      
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
              https://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/context
              https://www.springframework.org/schema/context/spring-context.xsd">
      
          <context:annotation-config/>
      
          <bean class="example.SimpleMovieCatalog">
              <qualifier value="main"/> 
      
              <!-- inject any dependencies required by this bean -->
          </bean>
      
          <bean class="example.SimpleMovieCatalog">
              <qualifier value="action"/> 
      
              <!-- inject any dependencies required by this bean -->
          </bean>
      
          <bean id="movieRecommender" class="example.MovieRecommender"/>
      
      </beans>
      
      ~~~

   3. 注意 当使用@Autowired这种通过类型注入的注解时候,最好还是使用id进行注入.避免发生未定义@qualifier而发生类型模糊问题.

#### @Resource

1. @Autowired可以放在字段上,构造方法上,多参数的方法上(set或者构造),而`@Resource` 仅可以放在字段或者单个参数的set方法上.
2. @Resource通过名称进行注入,@Autowired通过类型进行注入.

#### @Value

1. 使用方法

   1. ~~~java
      @Component
      public class MovieRecommender {
      
          private final String catalog;
      
          public MovieRecommender(@Value("${catalog.name}") String catalog) {
              this.catalog = catalog;
          }
      }
      @Configuration
      @PropertySource("classpath:application.properties")
      public class AppConfig { }
      // application.properties文件的内容为:
      catalog.name=MovieCatalog
      ~~~

   2. springboot默认会加载application.properties,所以这一步可以省略.

   3. 可以设置默认值

      1. ~~~java
         @Component
         public class MovieRecommender {
         
             private final String catalog;
         
             public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
                 this.catalog = catalog;
             }
         }
         ~~~

   4. 与@Value相关的类,此方法必须为static,可以查看所有通过@Value注入的值.

      1. ~~~java
         @Configuration
         public class AppConfig {
         
             @Bean
             public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
                 return new PropertySourcesPlaceholderConfigurer();
             }
         }
         ~~~

#### @PostConstruct和@PreDestory	

1. 与实现BeanPostProcess接口掌握容器类所有类声明周期不同,通过@PostConstruct和@PreDestroy掌握单个类的生命周期.

2. ~~~java
   public class CachingMovieLister {
   
       @PostConstruct
       public void populateMovieCache() {
           // populates the movie cache upon initialization...
       }
   
       @PreDestroy
       public void clearMovieCache() {
           // clears the movie cache upon destruction...
       }
   }
   ~~~

3. 注意JDK6-8支持这个两个注解,在JDK9中将他们从core中移除到其它地方,并在JDK11上彻底移除,但可以通过Maven依赖的方式支持.

## 类路径扫描与组件Component管理

### @Component和其它扩展构造注解

1. @Component是容器管理的通用组件,而@Controller,@Service,@Repository是在它基础上特殊化的组件.

   1. ~~~java
      @Target(ElementType.TYPE)
      @Retention(RetentionPolicy.RUNTIME)
      @Documented
      @Component 
      public @interface Service {
      
          // ...
      }
      ~~~

2. Spring可以自动的扫描组件给BeanDefinition,实例化后放入ApplicationContext.

### Spring在配置文件中开启扫描

1. ~~~java
   @Configuration
   @ComponentScan(basePackages = "org.example")
   public class AppConfig  {
       // ...
   }
   ~~~

2. 有时候为了简化也写成`@ComponentScan("org.example")`.

3. xml的方式

   1. ~~~xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
              https://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/context
              https://www.springframework.org/schema/context/spring-context.xsd">
      
          <context:component-scan base-package="org.example"/>
      
      </beans>
      ~~~

   2. 一般`<context:component-scan>` 开启了之后意味着`<context:annotation-config>` 已经开启了,不需要单独的配置.

   3. Springboot项目Applicationq启动类@SpringBootApplication注解已经包含了扫描注解,扫描的路径是当前启动类的包以及子包.

4. 可以自定义不需要扫到的包.

   1. ~~~java
      @Configuration
      @ComponentScan(basePackages = "org.example",
              includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
              excludeFilters = @Filter(Repository.class))
      public class AppConfig {
          // ...
      }
      ~~~

   2. ~~~xml
      <beans>
          <context:component-scan base-package="org.example">
              <context:include-filter type="regex"
                      expression=".*Stub.*Repository"/>
              <context:exclude-filter type="annotation"
                      expression="org.springframework.stereotype.Repository"/>
          </context:component-scan>
      </beans>
      ~~~

### 在组件中定义Bean的元数据

1. @Component可用定义含有@Bean的类,达到@Configuration一样的效果,本质它们是不同的@Configuration使用了CGLib 的增强,通过@Bean与其他的类产生联系,生成对应的代理调用对应的方法,代理和类的生命周期Spring 容器进行管理.
2. @Bean方法被static修饰:
   1. @Bean的方法被static修饰,因为此方法会容器生命周期的早起初始化,避免此方法与其他类产生联系.
   2. @Bean的方法永远都不会被拦截,即使是@Configuration修饰,由于技术原因,代理生成的子类只能重写非静态的方法.
   3. 如果@Bean的方法需要可重写,需要被@Configuration修饰并且就不能被private或者final修饰.

### 使用JSR330标准的注解

#### spring 3.0 支持JSR330的注解.

#### @Inject和@Named

1. 可用@Inject替换@Autowired

2. 有必须依赖的类 可用有@Named定义

3. ~~~java
   import javax.inject.Inject;
   import javax.inject.Named;
   
   public class SimpleMovieLister {
   
       private MovieFinder movieFinder;
   
       @Inject
       public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
           this.movieFinder = movieFinder;
       }
   
       // ...
   }
   ~~~

#### @Named和@ManagedBean

1. 可以替换@Component

2. ~~~java
   import javax.inject.Inject;
   import javax.inject.Named;
   
   @Named("movieListener")  // @ManagedBean("movieListener") could be used as well
   public class SimpleMovieLister {
   
       private MovieFinder movieFinder;
   
       @Inject
       public void setMovieFinder(MovieFinder movieFinder) {
           this.movieFinder = movieFinder;
       }
   
       // ...
   }
   ~~~

### 基于java的容器配置

#### `@Bean` and `@Configuration`

1. @Configuration修饰的类,暗示着对于Bean的定义是一种重要元素,需要优先进行初始化等处理.

2. 两种等价的定义

   1. ~~~java
      @Configuration
      public class AppConfig {
      
          @Bean
          public MyService myService() {
              return new MyServiceImpl();
          }
      }
      ~~~

   2. ~~~xml
      <beans>
          <bean id="myService" class="com.acme.services.MyServiceImpl"/>
      </beans>
      ~~~

3. 当@Bean使用在了@Component而没有用在@Configuration中的时候,仅作为工厂方法的机制,而不在具有CGLIb增强的属性.不被重定向到容器的生命周期管理.出现Bug时候更难排查.

#### `AnnotationConfigApplicationContext`

1. @Configuration和@Component注解的类,都会注册到AnnotationConfigApplicationContext容器中.

2. 调用方式

   1. ~~~java
      public static void main(String[] args) {
          ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
          MyService myService = ctx.getBean(MyService.class);
          myService.doStuff();
      }
      ~~~

   2. ~~~java
      public static void main(String[] args) {
          ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
          MyService myService = ctx.getBean(MyService.class);
          myService.doStuff();
      }
      ~~~

3. `register(Class<?>…)`方法.

   1. ```java
      public static void main(String[] args) {
          AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
          // 手动注册类
          ctx.register(AppConfig.class, OtherConfig.class);
          ctx.register(AdditionalConfig.class);
          ctx.refresh();
          MyService myService = ctx.getBean(MyService.class);
          myService.doStuff();
      }
      ```

4. 开启包扫描的三种方式 注解,xml和AnnotationConfigApplicationContext.scan()方法

   1. ~~~java
      @Configuration
      @ComponentScan(basePackages = "com.acme") 
      public class AppConfig  {
          // ...
      }
      ~~~

   2. ~~~xml
      <beans>
          <context:component-scan base-package="com.acme"/>
      </beans>
      ~~~

   3. ~~~java
      public static void main(String[] args) {
          AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
          ctx.scan("com.acme");
          ctx.refresh();
          MyService myService = ctx.getBean(MyService.class);
      }
      ~~~

#### @Bean的使用

1. 可以通过接口的方式实现Bean的配置

   1. ~~~java
      public interface BaseConfig {
      
          @Bean
          default TransferServiceImpl transferService() {
              return new TransferServiceImpl();
          }
      }
      
      @Configuration
      public class AppConfig implements BaseConfig {
      
      }
      ~~~

2. 返回类型可以是interface或者父类

   1. ~~~java
      @Configuration
      public class AppConfig {
      
          @Bean
          public TransferService transferService() {
              return new TransferServiceImpl();
          }
      }
      ~~~

   2. 这样定义只有TransferService接口对于容器来说是特殊的Bean,非惰性单例Bean根据调用顺序进行实例化,不同的类使用@Autowired TransferServiceImpl,可能导致返回的类型不同,当TransferService已经实例化后,可以向上转型为返回值的.而TransferServiceImpl只有在实例化之后,容器可能把它作为返回值.

3. @Bean可以定义生命周期方法

   1. ~~~java
      public class BeanOne {
      
          public void init() {
              // initialization logic
          }
      }
      
      public class BeanTwo {
      
          public void cleanup() {
              // destruction logic
          }
      }
      
      @Configuration
      public class AppConfig {
      
          @Bean(initMethod = "init")
          public BeanOne beanOne() {
              return new BeanOne();
          }
      
          @Bean(destroyMethod = "cleanup")
          public BeanTwo beanTwo() {
              return new BeanTwo();
          }
      }
      ~~~

4. @Bean可以与作用域@Scope,命名,别名,描述一起使用

   1. ~~~java
      @Configuration
      public class MyConfiguration {
      
          @Bean
          @Scope("prototype")
          public Encryptor encryptor() {
              // ...
          }
      }
      @Configuration
      public class AppConfig {
      
          @Bean("myThing")
          public Thing thing() {
              return new Thing();
          }
      }
      // 多个别名
      @Configuration
      public class AppConfig {
      
          @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
          public DataSource dataSource() {
              // instantiate, configure and return DataSource bean...
          }
      }
      @Configuration
      public class AppConfig {
      
          @Bean
          @Description("Provides a basic example of a bean")
          public Thing thing() {
              return new Thing();
          }
      }
      ~~~


#### @Configuration的使用

1. 实现多例调用

   ~~~java
   @Bean
   @Scope("prototype")
   public AsyncCommand asyncCommand() {
       AsyncCommand command = new AsyncCommand();
       // inject dependencies here as required
       return command;
   }
   
   @Bean
   public CommandManager commandManager() {
       // return new anonymous implementation of CommandManager with createCommand()
       // overridden to return a new prototype Command object
       return new CommandManager() {
           protected Command createCommand() {
               return asyncCommand();
           }
       }
   }
   ~~~

2. 不能生效的多例调用

   1. ~~~java
      @Configuration
      public class AppConfig {
      
          @Bean
          public ClientService clientService1() {
              ClientServiceImpl clientService = new ClientServiceImpl();
              clientService.setClientDao(clientDao());
              return clientService;
          }
      
          @Bean
          public ClientService clientService2() {
              ClientServiceImpl clientService = new ClientServiceImpl();
              clientService.setClientDao(clientDao());
              return clientService;
          }
      
          @Bean
          public ClientDao clientDao() {
              return new ClientDaoImpl();
          }
      }
      ~~~

   2. 原因:@Configuration的类都会作为CGLIB的子类,当子类调用父类的方法时候,会检测@Scoped作用域,默认为单例,则优先从已有的缓存中查找.更改是在clientDao方法上添加@Scope("prototype")

#### @import使用

1. 注入依赖

   1. ~~~java
      @Configuration
      public class ServiceConfig {
      
          @Bean
          public TransferService transferService(AccountRepository accountRepository) {
              return new TransferServiceImpl(accountRepository);
          }
      }
      
      @Configuration
      public class RepositoryConfig {
      
          @Bean
          public AccountRepository accountRepository(DataSource dataSource) {
              return new JdbcAccountRepository(dataSource);
          }
      }
      
      @Configuration
      @Import({ServiceConfig.class, RepositoryConfig.class})
      public class SystemTestConfig {
      
          @Bean
          public DataSource dataSource() {
              // return new DataSource
          }
      }
      
      public static void main(String[] args) {
          ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
          // everything wires up across configuration classes...
          TransferService transferService = ctx.getBean(TransferService.class);
          transferService.transfer(100.00, "A123", "C456");
      }
      ~~~

#### @ImportResource使用

1. ~~~java
   @Configuration
   @ImportResource("classpath:/com/acme/properties-config.xml")
   public class AppConfig {
   
       @Value("${jdbc.url}")
       private String url;
   
       @Value("${jdbc.username}")
       private String username;
   
       @Value("${jdbc.password}")
       private String password;
   
       @Bean
       public DataSource dataSource() {
           return new DriverManagerDataSource(url, username, password);
       }
   }
   ~~~

### 抽象的环境

#### @Profile使用

1. 定义:@Profile定义的类存活的时候,当前类才有资格进行初始化.

2. ~~~java
   @Configuration
   public class AppConfig {
   
       @Bean("dataSource")
       @Profile("development") 
       public DataSource standaloneDataSource() {
           return new EmbeddedDatabaseBuilder()
               .setType(EmbeddedDatabaseType.HSQL)
               .addScript("classpath:com/bank/config/sql/schema.sql")
               .addScript("classpath:com/bank/config/sql/test-data.sql")
               .build();
       }
   
       @Bean("dataSource")
       @Profile("production") 
       public DataSource jndiDataSource() throws Exception {
           Context ctx = new InitialContext();
           return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
       }
   }
   ~~~

3. 注意:重载有@Profile条件方法时候,@Profile依然有效,有时候会出现不一致的情况,导致方法永远无法调用,对用重名的方法,可以通过@Bean("name")不同的name进行限定.

4. 在xml中使用@Profile

   1. ~~~xml
      
      
      <beans profile="development"
          xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:jdbc="http://www.springframework.org/schema/jdbc"
          xsi:schemaLocation="...">
      
          <jdbc:embedded-database id="dataSource">
              <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
              <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
          </jdbc:embedded-database>
      </beans>
      
      
      <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:jdbc="http://www.springframework.org/schema/jdbc"
          xmlns:jee="http://www.springframework.org/schema/jee"
          xsi:schemaLocation="...">
      
          <!-- other bean definitions -->
      
          <beans profile="development">
              <jdbc:embedded-database id="dataSource">
                  <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
                  <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
              </jdbc:embedded-database>
          </beans>
      
          <beans profile="production">
              <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
          </beans>
      </beans>
      
      ~~~

   2. 激活@Profile

      1. 未激活会出现NoSuchBeanDefinitionException,直接通过ApplicationContext激活

         ~~~java
         AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
         ctx.getEnvironment().setActiveProfiles("development");
         ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
         ctx.refresh();
         ~~~

   #### `@PropertySource`  使用

   1. 可用通过Environment 判断source是否存在

      1. ~~~java
         ApplicationContext ctx = new GenericApplicationContext();
         Environment env = ctx.getEnvironment();
         boolean containsMyProperty = env.containsProperty("my-property");
         System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
         // getSource
         MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
         ~~~

   2. 定义:Spring环境中存在@PropertySource修饰的类才进行实例化.

###  `ApplicationContext`特有的能力

#### 相对于BeanFactory做了以下功能的增强

- Access to messages in i18n-style, through the `MessageSource` interface.
- Access to resources, such as URLs and files, through the `ResourceLoader` interface.
- Event publication, namely to beans that implement the `ApplicationListener` interface, through the use of the `ApplicationEventPublisher` interface.
- Loading of multiple (hierarchical) contexts, letting each be focused on one particular layer, such as the web layer of an application, through the `HierarchicalBeanFactory` interface.

#### MessageSource 国际化

#### Custom Event 发布订阅模式

### BeanFactory API

#### BeanFactory和Application比较

| Feature                                                      | `BeanFactory` | `ApplicationContext` |
| ------------------------------------------------------------ | ------------- | -------------------- |
| Bean instantiation/wiring                                    | Yes           | Yes                  |
| Integrated lifecycle management                              | No            | Yes                  |
| Automatic `BeanPostProcessor` registration                   | No            | Yes                  |
| Automatic `BeanFactoryPostProcessor` registration            | No            | Yes                  |
| Convenient `MessageSource` access (for internationalization) | No            | Yes                  |
| Built-in `ApplicationEvent` publication mechanism            | No            | Yes                  |

1. 能用Application不用BeanFactory.Application对应GenericApplicationContext,BeanFactory对应DefaultListableBeanFactory.

2. DefaultListableBeanFactory也可以注册BeanPostProcessor,但需要显示的调用.

   1. ~~~java
      DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
      // populate the factory with bean definitions
      
      // now register any needed BeanPostProcessor instances
      factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
      factory.addBeanPostProcessor(new MyBeanPostProcessor());
      
      // now start using the factory
      ~~~

# 2.Resources

## 介绍

​	java.net.URL没有标准的URL对ServletContext的资源的访问,所以Spring提供了Resource接口.

## Resource的接口

~~~java
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isReadable();
	// 防止内存泄露 正在读取时,其它的资源不能进行读取
    boolean isOpen();

    boolean isFile();

    URL getURL() throws IOException;

    URI getURI() throws IOException;

    File getFile() throws IOException;

    ReadableByteChannel readableChannel() throws IOException;

    long contentLength() throws IOException;

    long lastModified() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();
}

public interface InputStreamSource {

    InputStream getInputStream() throws IOException;
}
~~~

## Resource实现类

### Spring经常用到的实现类

- [`UrlResource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-implementations-urlresource)
- [`ClassPathResource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-implementations-classpathresource)
- [`FileSystemResource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-implementations-filesystemresource)
- [`PathResource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-implementations-pathresource)
- [`ServletContextResource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-implementations-servletcontextresource)
- [`InputStreamResource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-implementations-inputstreamresource)
- [`ByteArrayResource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-implementations-bytearrayresource)

### ResourceLoader

#### 定义:通过ResourceLoader获取Resource

~~~java
public interface ResourceLoader {

    Resource getResource(String location);

    ClassLoader getClassLoader();
}
ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext ();
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
~~~

#### 根据前缀获取不同的Resource

| Prefix     | Example                          | Explanation                                                  |
| ---------- | -------------------------------- | ------------------------------------------------------------ |
| classpath: | `classpath:com/myapp/config.xml` | Loaded from the classpath.                                   |
| file:      | `file:///data/config.xml`        | Loaded as a `URL` from the filesystem. See also [`FileSystemResource` Caveats](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources-filesystemresource-caveats). |
| https:     | `https://myserver/logo.png`      | Loaded as a `URL`.                                           |
| (none)     | `/data/config.xml`               | Depends on the underlying `ApplicationContext`.              |

### ResourcePatternResolver接口

#### 定义

​	解决了资源匹配的路径策略问题.Spring可以直接获取Resource下面的资源是通过此接口实现.

~~~java
public interface ResourcePatternResolver extends ResourceLoader {

    String CLASSPATH_ALL_URL_PREFIX = "classpath*:";

    Resource[] getResources(String locationPattern) throws IOException;
}
~~~

### ResourceLoaderAware接口

#### 定义

​	提供ResourceLoader的接口.

~~~java
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
~~~

#### 使用注意点

1. ApplicationContext就是一种,ResourceLoader,但是由于它比较大,一般不会将整类都返回.

### Resources作为依赖

#### 定义

​	当程序想要动态的加载资源,会将resource作为依赖.例如根据不同使用的角色加载不同的资源.

#### 通过xml

~~~java
package example;

public class MyBean {

    private Resource template;

    public setTemplate(Resource template) {
        this.template = template;
    }

    // ...
}
~~~

~~~xml
<bean id="myBean" class="example.MyBean">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
~~~

#### 通过注解

~~~java
@Component
public class MyBean {

    private final Resource template;

    public MyBean(@Value("${template.path}") Resource template) {
        this.template = template;
    }

    // ...
}
~~~

### Application Contexts and Resource Paths

#### 创建不同的ApplicationContext

1. 即使url更符合某一ApplicationContext,但是还是以实际调用的为准

   1. ~~~java
      ApplicationContext ctx =
          new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
      ~~~

   2. 其实"classpath:conf/appContext.xml"更符合ClassPathXmlApplicationContext.

### 资源路径使用通配符

~~~java
ApplicationContext ctx =
    new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
~~~

classpath*能够生效,因为ClassLoader的实现,如果application 服务有自己的ClassLoader,需要通过getClass().getClassLoader().getResources("<someFileInsideTheJar>")进行检查是否能够生效.

#### 常见的路径

```text
/WEB-INF/*-context.xml
com/mycompany/**/applicationContext.xml
file:C:/some/path/*-context.xml
classpath:com/mycompany/**/applicationContext.xml
```

# 3.检验,数据绑定和类型转换

## 校验

### 定义

​	验证不应该被绑定到web层，应该容易本地化，并且应该能够插入任何可用的验证器.基于此Spring的Validator在容器的每一层都能使用.

### 使用

~~~java
public class PersonValidator implements Validator {

    /**
     * This Validator validates only Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        // name 字段名 name.empty 是返回的信息
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
~~~

类相互依赖导致校验是的逻辑也要做对应的拷贝处理,可以将依赖类的校验类作为依赖,校验对应的类.

~~~java
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * This Validator validates Customer instances, and any subclasses of Customer too
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
~~~

## BeanWrapper

1. 虽然不经常使用BeanWarpper,但是可以通过它实现数据的绑定.而且通过此类实现PropertyEditor经常被用到.

2. ~~~java
   public class Company {
   
       private String name;
       private Employee managingDirector;
   
       public String getName() {
           return this.name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public Employee getManagingDirector() {
           return this.managingDirector;
       }
   
       public void setManagingDirector(Employee managingDirector) {
           this.managingDirector = managingDirector;
       }
   }
   
   public class Employee {
   
       private String name;
   
       private float salary;
   
       public String getName() {
           return this.name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public float getSalary() {
           return salary;
       }
   
       public void setSalary(float salary) {
           this.salary = salary;
       }
   }
   BeanWrapper company = new BeanWrapperImpl(new Company());
   // setting the company name..
   company.setPropertyValue("name", "Some Company Inc.");
   // ... can also be done like this:
   PropertyValue value = new PropertyValue("name", "Some Company Inc.");
   company.setPropertyValue(value);
   
   // ok, let's create the director and tie it to the company:
   BeanWrapper jim = new BeanWrapperImpl(new Employee());
   jim.setPropertyValue("name", "Jim Stravinsky");
   company.setPropertyValue("managingDirector", jim.getWrappedInstance());
   
   // retrieving the salary of the managingDirector through the company
   Float salary = (Float) company.getPropertyValue("managingDirector.salary");
   ~~~

3. `PropertyEditor` :当例如String类型的时间,默认转换成可读的'2007-14-09'还是原始的时间格式,可以通过注册PropertyEditor来进行定义.Controller的属性绑定也通过PropertyEditor来实现.

4. 可以自定义`PropertyEditor` ,并通过`PropertyEditorRegistrar`进行注册生效.具体见

   [官网]: https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation	"官网"

## 类型转换

### Converter接口实现

~~~java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

    T convert(S source);
}

package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

    public Integer convert(String source) {
        return Integer.valueOf(source);
    }
}
~~~

### `ConverterFactory`

1. 如果想要将类型转换应用到整个类层次结构,需要使用ConverterFactory.

2. ~~~java
   package org.springframework.core.convert.converter;
   // S 转换到R
   public interface ConverterFactory<S, R> {
   
       <T extends R> Converter<S, T> getConverter(Class<T> targetType);
   }
   ~~~

3. ~~~java
   package org.springframework.core.convert.support;
   
   final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {
   
       public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
           return new StringToEnumConverter(targetType);
       }
   
       private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {
   
           private Class<T> enumType;
   
           public StringToEnumConverter(Class<T> enumType) {
               this.enumType = enumType;
           }
   
           public T convert(String source) {
               return (T) Enum.valueOf(this.enumType, source.trim());
           }
       }
   }
   ~~~

### `GenericConverter`

1. GenericConverter虽然没有Converter功能强大,但在简单类型的转换上更加的灵活.

2. ~~~java
   package org.springframework.core.convert.converter;
   
   public interface GenericConverter {
   
       public Set<ConvertiblePair> getConvertibleTypes();
   
       Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
   }
   ~~~

3. `ConditionalGenericConverter` 条件匹配后才转换

   1. ~~~java
      public interface ConditionalConverter {
      
          boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);
      }
      
      public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
      }
      ~~~

### ConversionService 定义

1. ConversionService 应该是一个无状态的,不同的容器都可定义ConversionService .

2. ~~~xml
   <bean id="conversionService"
           class="org.springframework.context.support.ConversionServiceFactoryBean">
       <property name="converters">
           <set>
               <bean class="example.MyCustomConverter"/>
           </set>
       </property>
   </bean>
   ~~~

3. strings, numbers, enums, collections, maps, and other common types都可以进行转换.

4. 代码的方式使用ConversionService.

~~~java
@Service
public class MyService {

    public MyService(ConversionService conversionService) {
        this.conversionService = conversionService;
    }

    public void doIt() {
        this.conversionService.convert(...)
    }
}

// 使用 TypeDescriptor 简化操作
DefaultConversionService cs = new DefaultConversionService();
// 将集合中的Integer类型转换为String
List<Integer> input = ...
cs.convert(input,
    TypeDescriptor.forObject(input), // List<Integer> type descriptor
    TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
~~~

## 字段格式化

定义:当需要类型转变的时候,使用ConversionService,但是当例如我们需要根据不同的环境格式化String的时候,可以用Spring3提供的Formatter或者PropertyEditor.

### `Formatter` SPI

1. ~~~java
   package org.springframework.format;
   
   public interface Formatter<T> extends Printer<T>, Parser<T> {
   }
   
   public interface Printer<T> {
   
       String print(T fieldValue, Locale locale);
   }
   
   import java.text.ParseException;
   
   public interface Parser<T> {
   
       T parse(String clientValue, Locale locale) throws ParseException;
   }
   ~~~

2. 其它格式化的工具:The `number` package provides `NumberStyleFormatter`, `CurrencyStyleFormatter`, and `PercentStyleFormatter` to format `Number` objects that use a `java.text.NumberFormat`. The `datetime` package provides a `DateFormatter` to format `java.util.Date` objects with a `java.text.DateFormat`.

3. 实现时间格式化的例子

   1. ~~~java
      package org.springframework.format.datetime;
      
      public final class DateFormatter implements Formatter<Date> {
      
          private String pattern;
      
          public DateFormatter(String pattern) {
              this.pattern = pattern;
          }
      
          public String print(Date date, Locale locale) {
              if (date == null) {
                  return "";
              }
              return getDateFormat(locale).format(date);
          }
      
          public Date parse(String formatted, Locale locale) throws ParseException {
              if (formatted.length() == 0) {
                  return null;
              }
              return getDateFormat(locale).parse(formatted);
          }
      
          protected DateFormat getDateFormat(Locale locale) {
              DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
              dateFormat.setLenient(false);
              return dateFormat;
          }
      }
      ~~~

### 注解的方式格式字段

1. ~~~java
   package org.springframework.format;
   
   public interface AnnotationFormatterFactory<A extends Annotation> {
   
       Set<Class<?>> getFieldTypes();
   
       Printer<?> getPrinter(A annotation, Class<?> fieldType);
   
       Parser<?> getParser(A annotation, Class<?> fieldType);
   }
   ~~~

2. AnnotationFormatterFactory通过绑定@NumberFormat 实现自定义的格式化

   1. ~~~java
      public final class NumberFormatAnnotationFormatterFactory
              implements AnnotationFormatterFactory<NumberFormat> {
      
          public Set<Class<?>> getFieldTypes() {
              return new HashSet<Class<?>>(asList(new Class<?>[] {
                  Short.class, Integer.class, Long.class, Float.class,
                  Double.class, BigDecimal.class, BigInteger.class }));
          }
      	// NumberFormat
          public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
              return configureFormatterFrom(annotation, fieldType);
          }
      
          public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
              return configureFormatterFrom(annotation, fieldType);
          }
      
          private Formatter<Number> configureFormatterFrom(NumberFormat annotation, Class<?> fieldType) {
              if (!annotation.pattern().isEmpty()) {
                  return new NumberStyleFormatter(annotation.pattern());
              } else {
                  Style style = annotation.style();
                  if (style == Style.PERCENT) {
                      return new PercentStyleFormatter();
                  } else if (style == Style.CURRENCY) {
                      return new CurrencyStyleFormatter();
                  } else {
                      return new NumberStyleFormatter();
                  }
              }
          }
      }
      ~~~

   2. ~~~java
      public class MyModel {
      
          @NumberFormat(style=Style.CURRENCY)
          private BigDecimal decimal;
      }
      ~~~

3. 标准化的注解

   1. org.springframework.format.annotation 包下的@NumberFormat 格式化数字, `@DateTimeFormat`格式化时间.

   2. ~~~java
      public class MyModel {
      
          @DateTimeFormat(iso=ISO.DATE)
          private Date date;
      }
      ~~~

4. 可以通过`FormatterRegistry`  注册自己的格式化类.详见官网.

### 	全局的时间格式化

1. 注解的方式

   ~~~java
   @Configuration
   public class AppConfig {
   
       @Bean
       public FormattingConversionService conversionService() {
   
           // Use the DefaultFormattingConversionService but do not register defaults
           DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);
   
           // Ensure @NumberFormat is still supported
           conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());
   
           // Register JSR-310 date conversion with a specific global format
           DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
           registrar.setDateFormatter(DateTimeFormatter.ofPattern("yyyyMMdd"));
           registrar.registerFormatters(conversionService);
   
           // Register date conversion with a specific global format
           DateFormatterRegistrar registrar = new DateFormatterRegistrar();
           registrar.setFormatter(new DateFormatter("yyyyMMdd"));
           registrar.registerFormatters(conversionService);
   
           return conversionService;
       }
   }
   ~~~

2. 就是将DefaultFormattingConversionService默认支持的格式化方式去掉,给有自己定义的时间格式化器.

3. xml的方式

   ~~~xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd>
   
       <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
           <property name="registerDefaultFormatters" value="false" />
           <property name="formatters">
               <set>
                   <bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
               </set>
           </property>
           <property name="formatterRegistrars">
               <set>
                   <bean class="org.springframework.format.datetime.standard.DateTimeFormatterRegistrar">
                       <property name="dateFormatter">
                           <bean class="org.springframework.format.datetime.standard.DateTimeFormatterFactoryBean">
                               <property name="pattern" value="yyyyMMdd"/>
                           </bean>
                       </property>
                   </bean>
               </set>
           </property>
       </bean>
   </beans>
   ~~~

### Java Bean的有效性验证

~~~java
public class PersonForm {

    @NotNull
    @Size(max=64)
    private String name;

    @Min(0)
    private int age;
}
~~~

一般会根据引入的依赖API进行使用,例如[Hibernate Validator](https://hibernate.org/validator/),如果将验证作为Spring Bean,可以官网学习.

# 4.Spring表达式语言(SpEL)

定义:可以在编译时期查询或者操作对象.

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')"); 
String message = (String) exp.getValue();
```

应用场景百度了一大圈也不知道常用的场景到底是啥,暂时先不学习.

# 5.面向切面编程

## AOP概念

- 切面:想要织入的逻辑
- 切入点:程序被织入的地方.
- 通知:特定的连接点上采取的动作.
- 切点:一个表达式,表达切入点的位置
- 介绍:代表类型声明额外的方法或字段.
- 目标对象:被织入通知的对象.
- AOP代理
- 织入:连接切面和目标对象.

## 支持@AspectJ

### 注解的方式和xlm的方式开启切面配置

~~~java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
~~~

~~~xml
<aop:aspectj-autoproxy/>
~~~

### 定义一个切面

1. ~~~java
   <bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
       <!-- configure properties of the aspect here -->
   </bean>
   ~~~

2. ~~~java
   package org.xyz;
   import org.aspectj.lang.annotation.Aspect;
   
   @Aspect
   public class NotVeryUsefulAspect {
   
   }
   ~~~

3. 注意使用,可用通过@Bean和@Configuration注册一个切面,也可以使用类路径扫描,**但是切面不足以类的自动监测**,需要格外的@Component注解.

### 定义一个切点

1. 切点是描述切面需要加入的位置.

2. ~~~java
   @Pointcut("execution(* transfer(..))") // the pointcut expression
   private void anyOldTransfer() {} // the pointcut signature
   ~~~

3. 切点支持的标识符

   1. execution:

4. 一般JDK的代理只有public方法的代理,才有可能被拦截.而CGLib的代理可以是public和protect方法,如果切点严格定义是公共方法,如果想被拦截,protectd的方法需要做相应的改变.

5. 经常用到的切面表达

   1. 任何的public 方法时执行

      ```java
       execution(public * *(..))
      ```

   2. 任何以set开头的方法时执行

      ~~~java
       execution(* set*(..))
      ~~~

   3. service包下执行

      ```java
       execution(* com.xyz.service.*.*(..))
      ```

   4. service包以及子包

      ~~~java
      execution(* com.xyz.service..*.*(..))
      ~~~

   5. 任何切面涉及到service包,只针对Spring的AOP.

      ```java
      within(com.xyz.service.*)
      ```

   6. 任何切面的代理实现了AccountService接口的时候

      ~~~java
      this(com.xyz.service.AccountService)
      ~~~

   7. 任何切面的目标对象实现了AccountService接口

      ```java
      target(com.xyz.service.AccountService)
      ```

   8. 任何切面使用到了一个参数,而且运行是传递的参数类型是Serializable,注意它与execution(* *(java.io.Serializable))不同,这个代表的匹配到生命类型为Serializable的方法.

      ```java
      args(java.io.Serializable)
      ```

   9. 任何切面方法的目标对象使用了@Transactional注解

      ```java
      @target(org.springframework.transaction.annotation.Transactional)
      ```

   10. 任何切面方法目标对象的生命类型使用了@Transactional注解

       ```java
       @within(org.springframework.transaction.annotation.Transactional)
       ```

   11. 任何切面方法执行的方法使用了@Transactional注解

       ```java
       @annotation(org.springframework.transaction.annotation.Transactional)
       ```

   12. 任何切面方法有单个参数,并且运行时候类型有@Classified注解

       ```java
        @args(com.xyz.security.Classified)
       ```

   13. 任何切面方法有一个Bean叫做tradeService

       ```java
        bean(tradeService)
       ```

   14. 任何切面方法有一个Bean名称匹配*Service

       ```java
       bean(*Service)
       ```

6. 切点的原则是定位范围尽量的窄.写好一个切点必须包含Kinded 和Scoping 

   1. Kinded designators select a particular kind of join point: `execution`, `get`, `set`, `call`, and `handler`.
   2. Scoping designators select a group of join points of interest (probably of many kinds): `within` and `withincode`
   3. Contextual designators match (and optionally bind) based on context: `this`, `target`, and `@annotation`

### 定义一个通知

#### 定义

通知与切入点表达式相关联，并在切入点匹配的方法执行之前、之后或前后运行。

#### 前置通知

1. ~~~java
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   
   @Aspect
   public class BeforeExample {
   
       @Before("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
       public void doAccessCheck() {
           // ...
       }
   }
   // 换成另一种切点的表达 两种等价
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   
   @Aspect
   public class BeforeExample {
   
       @Before("execution(* com.xyz.myapp.dao.*.*(..))")
       public void doAccessCheck() {
           // ...
       }
   }
   ~~~

#### After Returning Advice

1. 当匹配的方法正常执行有返回值时候执行

   1. 

   ~~~java
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.AfterReturning;
   
   @Aspect
   public class AfterReturningExample {
   
       @AfterReturning("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
       public void doAccessCheck() {
           // ...
       }
   }
   ~~~

2. 可以通过返回值确定方法的执行,但是返回值需要与对应方法返回值类型匹配.

   1. ~~~java
      import org.aspectj.lang.annotation.Aspect;
      import org.aspectj.lang.annotation.AfterReturning;
      
      @Aspect
      public class AfterReturningExample {
      
          @AfterReturning(
              pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
              returning="retVal")
          public void doAccessCheck(Object retVal) {
              // ...
          }
      }
      ~~~

#### After Throwing Advice

1. 当匹配的方法正常执行并抛出异常的时候执行

   1. ~~~java
      import org.aspectj.lang.annotation.Aspect;
      import org.aspectj.lang.annotation.AfterThrowing;
      
      @Aspect
      public class AfterThrowingExample {
      
          @AfterThrowing("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
          public void doRecoveryActions() {
              // ...
          }
      }
      ~~~

2. 可以通过定义方法返回的参数判断执行

   1. ~~~java
      @Aspect
      public class AfterThrowingExample {
      
          @AfterThrowing(
              pointcut="com.xyz.myapp.CommonPointcuts.dataAccessOperation()",
              throwing="ex")
          public void doRecoveryActions(DataAccessException ex) {
              // ...
          }
      }
      ~~~

   2. 返回值参数的名称必须是ex

#### After (Finally) Advice

1. 只要匹配的方法执行了(不管是否有返回值返回)就执行,必须考虑匹配的方法是正常执行还是异常.

   1. ~~~java
      import org.aspectj.lang.annotation.Aspect;
      import org.aspectj.lang.annotation.After;
      
      @Aspect
      public class AfterFinallyExample {
      
          @After("com.xyz.myapp.CommonPointcuts.dataAccessOperation()")
          public void doReleaseLock() {
              // ...
          }
      }
      ~~~

   2. @After是只有匹配的方法有返回值才执行.

#### Around Advice

1. ~~~java
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Around;
   import org.aspectj.lang.ProceedingJoinPoint;
   
   @Aspect
   public class AroundExample {
   
       @Around("com.xyz.myapp.CommonPointcuts.businessService()")
       public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
           // start stopwatch
           Object retVal = pjp.proceed();
           // stop stopwatch
           return retVal;
       }
   }
   ~~~

#### 通知的参数

1. 也可以根据方法的参数匹配决定是否执行通知

   1. ~~~java
      @Pointcut("com.xyz.myapp.CommonPointcuts.dataAccessOperation() && args(account,..)")
      private void accountDataAccessOperation(Account account) {}
      
      @Before("accountDataAccessOperation(account)")
      public void validateAccount(Account account) {
          // ...
      }
      ~~~

2. 复杂条件的通知

   1. ~~~java
      @Before(value="com.xyz.lib.Pointcuts.anyPublicMethod() && target(bean) && @annotation(auditable)",
              argNames="bean,auditable")
      public void audit(JoinPoint jp, Object bean, Auditable auditable) {
          AuditCode code = auditable.value();
          // ... use code, bean, and jp
      }
      ~~~

#### 通知的优先级

1. 可以通过@Order定义@Aspect顺序.
2. Spring Framework 5.2.7之后相同条件先通知的执行的顺序是:@Around`, `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing.

#### 说明(Introductions)

1. 可以定义执行的条件是通知的对象实现哪些接口或者直接定义依赖对象.

2. 可以使用@DeclareParents

   1. ~~~java
      @Aspect
      public class UsageTracking {
      
          @DeclareParents(value="com.xzy.myapp.service.*+", defaultImpl=DefaultUsageTracked.class)
          public static UsageTracked mixin;
      
          @Before("com.xyz.myapp.CommonPointcuts.businessService() && this(usageTracked)")
          public void recordUsage(UsageTracked usageTracked) {
              usageTracked.incrementUseCount();
          }
      
      }
      ~~~

### 以xml方式定义一个通知

1. 与注解方式相对应 详见官网.

## 选择AOP的声明风格

如果通知是在Spring Bean上执行,Spring AOP是一个较好的选择,如果影响的是容器,使用AspectJ更好.

决定使用Spring AOP ,选择xml还是@AspectJ:一般作为服务配置一部分时,使用xml的方式会更好,xml不与代码融合独立的优势返回出来,当最为单元方法,@Aspect更好一些.

## 代理机制

### 定义

​	动态代理分为JDK代理和CGLIB代理,JDK代理是基于接口的,而CGLib是基于父类的.CGlib不能代理final修饰的方法.

​    如果非要使用CGLIB代理,需要配置开启

~~~xml
<aop:config proxy-target-class="true">
    <!-- other beans defined here... -->
</aop:config>
~~~

### 理解代理

1. 生成代理对象去执行对应的方法,但是当调用this.method()时候,会直接由对象本体执行而不是代理执行.但需要尽量避免.

# 6.Spring AOP APIs

## Spring 中的Pointcut API

### 概念

1. Pointcut接口

   ~~~java
   public interface Pointcut {
   
       ClassFilter getClassFilter();
   
       MethodMatcher getMethodMatcher();
   }
   ~~~

2. ClassFilter是否符合切点定义的一系列Class

   1. ~~~java
      public interface ClassFilter {
      
          boolean matches(Class clazz);
      }
      ~~~

3. MethodMatcher  更加的重要

   ~~~java
   public interface MethodMatcher {
   	// 匹配切点的目标对象的类型 避免每个方法都进行验证
       boolean matches(Method m, Class<?> targetClass);
   	// 是否在运行
       boolean isRuntime();
   	// 方法的参数匹配 验证每个方法
       boolean matches(Method m, Class<?> targetClass, Object... args);
   }
   ~~~

## Spring通知的API

### 重写通知

1. 可以自定义循环,前置,后置通知等

   1. 循环通知  实现 MethodInterceptor 
   2. 前置通知  实现BeforeAdvice 
   3. 异常通知 实现ThrowsAdvice 
   4. After Return通知 实现AfterReturningAdvice 

2. 代码

   1. ~~~java
      public interface MethodInterceptor extends Interceptor {
      
          Object invoke(MethodInvocation invocation) throws Throwable;
      }
      
      
      public class DebugInterceptor implements MethodInterceptor {
      
          public Object invoke(MethodInvocation invocation) throws Throwable {
              System.out.println("Before: invocation=[" + invocation + "]");
              Object rval = invocation.proceed();
              System.out.println("Invocation returned");
              return rval;
          }
      }
      ~~~

## Spring 通知者(Advisor)的API

### 使用`ProxyFactoryBean`创建AOP Proxies

​	向FactoryBean一样,Spring支持这种代理并管理生成代理的生命周期

### 代理Bean的属性

1. 继承ProxyConfig类

~~~java
package org.springframework.aop.framework;

import java.io.Serializable;
import org.springframework.util.Assert;

public class ProxyConfig implements Serializable {
    private static final long serialVersionUID = -8409359707199703185L;
    private boolean proxyTargetClass = false;
    private boolean optimize = false;
    boolean opaque = false;
    boolean exposeProxy = false;
    private boolean frozen = false;

~~~

2. 重要属性介绍
   1. proxyTargetClass: 如果设置成true使用CGLIB代理.
   2. optimize: 控制是否对通过CGLIB创建的代理应用积极优化,一般不适用值针对CGLIB生效
   3. frozen:如果代理配置被冻结，则不再允许对该配置进行更改。
   4. exposeProxy:是否将代理暴露给ThreadLocal

### 	代理生成

1. 如果对象实现了接口,则使用jdk,如果没有实现接口才会使用CGLIB生成代理,即使proxyTargetClass设置成了false.如果设置成为了ture,则使用CGLIB生成代理.

2. 配置personTarget的代理

   1. ~~~xml
      <bean id="personTarget" class="com.mycompany.PersonImpl">
          <property name="name" value="Tony"/>
          <property name="age" value="51"/>
      </bean>
      
      <bean id="myAdvisor" class="com.mycompany.MyAdvisor">
          <property name="someProperty" value="Custom string property value"/>
      </bean>
      
      <bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor">
      </bean>
      
      <bean id="person"
          class="org.springframework.aop.framework.ProxyFactoryBean">
          <property name="proxyInterfaces" value="com.mycompany.Person"/>
      
          <property name="target" ref="personTarget"/>
          <property name="interceptorNames">
              <list>
                  <value>myAdvisor</value>
                  <value>debugInterceptor</value>
              </list>
          </property>
      </bean>
      ~~~

   2. ~~~java
      // 获取代理对象
      Person person = (Person) factory.getBean("person");
      ~~~

   3. 可以通过匿名内部类隐藏代理和目标的区别.只有一种类型的Person避免了混淆.


## 简洁代理

### 继承父类 重写代理的方法

1. ~~~xml
   <bean id="txProxyTemplate" abstract="true"
           class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
       <property name="transactionManager" ref="transactionManager"/>
       <property name="transactionAttributes">
           <props>
               <prop key="*">PROPAGATION_REQUIRED</prop>
           </props>
       </property>
   </bean>
   ~~~

2. ~~~xml
   <bean id="mySpecialService" parent="txProxyTemplate">
       <property name="target">
           <bean class="org.springframework.samples.MySpecialServiceImpl">
           </bean>
       </property>
       <property name="transactionAttributes">
           <props>
               <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
               <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
               <prop key="load*">PROPAGATION_REQUIRED,readOnly</prop>
               <prop key="store*">PROPAGATION_REQUIRED</prop>
           </props>
       </property>
   </bean>
   ~~~

## 通过`ProxyFactory`生成代理

### 目的

​	在不依赖Spring IOC使用Spring AOP.但是不推荐这种使用方式.

~~~java
ProxyFactory factory = new ProxyFactory(myBusinessInterfaceImpl);
factory.addAdvice(myMethodInterceptor);
factory.addAdvisor(myAdvisor);
MyBusinessInterface tb = (MyBusinessInterface) factory.getProxy();
~~~

## 操作Adviced 对象

通过org.springframework.aop.framework.Advised接口实现.

~~~java
Advisor[] getAdvisors();

void addAdvice(Advice advice) throws AopConfigException;

void addAdvice(int pos, Advice advice) throws AopConfigException;

void addAdvisor(Advisor advisor) throws AopConfigException;

void addAdvisor(int pos, Advisor advisor) throws AopConfigException;

int indexOf(Advisor advisor);

boolean removeAdvisor(Advisor advisor) throws AopConfigException;

void removeAdvisor(int index) throws AopConfigException;

boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException;

boolean isFrozen();
~~~

## TargetSource

### 定义

​	此接口返回所有与切点有关的target对象.

### 常用的TargetSource

1. Hot-swappable Target Sources
   1. 通过org.springframework.aop.target.HotSwappableTargetSource实现,存在的目的是允许切换AOP代理的目标，同时让调用者保留对它的引用。
   
2. Pooling Target Sources

3. Prototype Target Sources

4. ThreadLocal Target Sources
   1. 如果想为每次请求创建一个对象,使用ThreadLocal Target Source
   
   2. ~~~xml
      <bean id="threadlocalTargetSource" class="org.springframework.aop.target.ThreadLocalTargetSource">
          <property name="targetBeanName" value="businessObjectTarget"/>
      </bean>
      ~~~































