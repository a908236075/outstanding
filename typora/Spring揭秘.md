# Spring揭秘

## 第一章Spring框架总体结构

1. IOC容器,AOP, DAO JDBC和事物管理,ORM Mybatis,SpringMVC.
2. ![image-20201209100946514](.\picture\spring揭秘\image-20201209100946514.png)

## 第二章 IOC的基本概念

### IOC:控制反转

1. 由IOC Service Provider统一管理,将被依赖的对象注入到注入对象中.

### Bean注入方式

#### 构造方法注入:

- ![image-20201209134452067](.\picture\spring揭秘\image-20201209134452067.png)
- 好处是随着对象一起创建.缺点是当依赖对象比较多的时候,参数会比较长.通过反射构造对象的时候,对相同类型的参数处理比较困难.构造方法无法被继承,无法设置默认值.参数数量的变动可能会引起维护上面的不变.

```xml
<bean id="personDao" class="com.fredia.service.impl.PersonDaoBean"></bean >  
<!--构造器方式注入-->  
<bean id="personService" class="com.fredia.service.impl.PersonServiceBean">  
    <constructor-arg index="0" type="cn.glzaction.service.impl.PersonDaoBean" ref="personDao"/>  
    <constructor-arg index="1" type="java.lang.String" value="glzaction"/>  
    <constructor-arg index="2" type="java.util.List">  
        <list>  
            <value>list1</value>  
            <value>list2</value>  
            <value>list3</value>  
        </list>  
    </constructor-arg>  
</bean>  

```

#### Set方法注入

- ![image-20201209134553435](.\picture\spring揭秘\image-20201209134553435.png)

- 应为set方法可以命名,会比构造方法好一些,setter方法可以别继承,可以设置默认值,缺点是无法再构造完成后马上进入就绪的状态.初始化必须依赖的类使用构造方法注入,可选择的类使用set方法注入,有时候用set方法注入解决循环依赖的问题.

- ```xml
  <bean id="personDao" class="com.fredia.service.impl.PersonDaoBean">  
      <property name="name" type="java.lang.String" value="glzaction"/>  
      <property name="id" type="java.lang.Integer" value="1"/>  
      <property name="list" type="java.util.List">  
          <list>  
              <value>list1</value>  
              <value>list2</value>  
              <value>list3</value>  
          </list>  
      </property>  
      <property name="map" type="java.util.Map">  
          <map>  
              <entry key="key1" value="value1"></entry>  
              <entry key="key2" value="value2"></entry>  
          </map>  
      </property>  
  </bean>  
  ```

- 接口的注入:不推荐使用,通过实现响应的接口来实现注入.

### 对IOC真正的理解

1. spring是单例模式的,如果没有IOC,每一个都要编写单例的方法.
2. 对于Bean的**依赖管理**,如果类A依赖B,C,D类,不需要一个一个的new这样耦合在代码的方式,通过IOC可以实现Bean的管理和解耦.

### Autowired注解

1. 根据Autowired放置的位置不同,底层使用的注入方法不同.

2. Autowired放在构造方法上，利用构造器注入，类初始化使就会创建依赖。

3. Autowired放在set方法上，setter方法注入，setter代码冗长，调用的时候才会被实例化。不能将属性设置为final。

4. Autowired放在类上，用的变量注入，通过优先通过类型(然后通过类名)到BeanFactory里面寻找对应类型的.缺点:可能发生空指针异常,解决办法是用注解(Service,Component等)或者配置文件的方式将类进行实例化.除了反射来提供需要的类,单元测试时候无法复用,需要启动整个容器.

### IOC Service Provider

1. 业务对象的构建管理
2. 业务对象之间的依赖绑定
  1. **直接编码的方式**:在容器中直接用代码管理关系.所有方式的最终方式.

    - ![image-20201209145322335](.\picture\spring揭秘\image-20201209145322335.png)
    - 通过代码可以看到先注册了两个类,等需要之前注册过的类,直接从容器中获取.
  2. **配置文件的方式**,xml的方式最常见.
  3. **注解的方式**,其实注最终还是编码的方式来确定注入的关系.
  4. Depend on 使用:

     1. ~~~xml
        <bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
            <property name="manager" ref="manager" />
        </bean>
        
        <bean id="manager" class="ManagerBean" />
        <bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
        ~~~

     2. 当依赖关系并不十分直接的时候可以用depend-on属性,多依赖.会在初始化beanOne之前初始化depends-on的类.


## 第四章 Spring的IOC容器之BeanFactory

### spring Ioc容器和Ioc provider的关系

![image-20201209151641139](.\picture\spring揭秘\image-20201209151641139.png)

### spring提供了两种容器类型:BeanFactory和ApplicationContext

- BeanFactory:基础类型IOC容器,提供完整的Ioc服务支持.默认采用**延迟初始化策略(懒加载)**.启动时间快,需要的系统资源少.
- ApplicationContext:在BeanFactory上构建,;对其进行了功能上的升级,比如事件发布,国际化信息支持等**.默认启动时完成所有初始化**,需要更多的系统资源.
- ![image-20201209152151032](.\picture\spring揭秘\image-20201209152151032.png)

### BeanFactory,BeanDefinitionRegister以及DefaultListtableBeanFactory的关系

1. BeanFactory 只是一个接口，我们最终需要一个该接口的实现来进行实际的Bean的管理， Default-
   ListableBeanFactory 就是这么一个比较通用的 BeanFactory 实现类。 DefaultListableBean-
   Factory 除了间接地实现了 BeanFactory 接口，还实现了 BeanDefinitionRegistry 接口，该接口才
   是在 BeanFactory 的实现中担当Bean注册管理的角色。基本上， BeanFactory 接口只定义如何访问容
   器内管理的Bean的方法，各个 BeanFactory 的具体实现类负责具体Bean的注册以及管理工作。
   BeanDefinitionRegistry 接口定义抽象了Bean的注册逻辑。通常情况下，具体的 BeanFactory 实现
   类会实现这个接口来管理Bean的注册。它们之间的关系如图4-3所示

![image-20201209160039061](.\picture\spring揭秘\image-20201209160039061.png)

### BeanFactory的对象注册与依赖绑定方式

- 直接编码的方式:BeanDefinitionRegistry接口定义Bean的注册逻辑.每一个受管理的对象在容器中都会有一个BeanDefinition(instance)的实例与之相对应.
- 外部配置文件方式
  - 一般用BeanDefinitionReader来读取配置文件.
  - properties
  - xml    **spring读取xml配置文件的过程?**
    -  set方法,构造函数方法,
    -  list,set,map 复杂数据结构的封装.各种标签的介绍.
    -  byName和byType自动绑定,不用我们手写class类,省去了一些代码量,但是类型唯一时候才能保证成功.
    -  lazy-init 主要针对application容器设置延时初始化.
- 注解方式.注解注入其实是使用注解的方式进行构造器注入或者set注入。使用哪种方式注入,通过位置来判断,如果@autowired在构造方法上就是使用了构造器注入的.**这里需要区分Bean注入的方式和Bean注册方式的区别.**注解的方式其实是Bean注册方式的一种.注册是管理的类之前的关系,注入关注的是类的生成.
  - Bean的scope
    - 单例 与IOC容器拥有相同的寿命.
    - 多例  容器为每次请求创建一个对象 ,随后不在管理他的生命周期.
    - request,session,global session:request是多例的一个特例,只是拥有了具体的使用场景.
  - FactoryBean本质上一个Bean,只不过这种类型的bean本身,就是生产对象的工厂.不要与BeanFactory相混淆.

1. Bean的Scope:配置中的Bean可以看多是模板,容器会根据他们构造对象,的说那是要创建多少个,构造完成后对象实例存活多久,则是由容器根据Bean的scope语义来决定的.

2. 注意:GOF的单例模式和spring的Singleton模式是不同的,标记为Singleton的bean是由容器来保证这类的bean在同一个容器中只存在一个共享实例;Singleton模式则是保证同一个ClassLoader中只存在一个这种类型的实例.而设计模式中，我们是对构造方法私有化，进行单例模式，用户从而不能new多个实例。

3. 单例:容器中只存在一个共享的实例.存活时间:从容器启动,到它第一次被请求实例化开始,只要是容器不销毁或者退出,改Bean就会一直存活.多例:每次请求都会创建一个全新的对象,只负责创建,后期的对象的生命周期由请求方来维护.

4. request,session,global session只适用于web应用程序.

5. 小心prototype(多例的陷阱),有时候即使我们配置了多例,但是程序取对象的时候只取同一个对象,这是因为引用不当,每次请求没有向IOC的容器中获取对象,解决方法如下:

   1. 需要配置lookup-method方法注入的方式.

   ```xml
   <bean id="persister" class="">
   	<lookup-method name  bean></lookup-method>
   </bean>
   ```

   2. 使用 @Lookup注解的方式

      1. ~~~java
         @Component
         public class Write{
         
            @Autowired
             private Learn learn;
             private String name;
         
             public String getName() {
                 return name;
             }
         
             public void setName(String name) {
                 this.name = name;
             }
         
              @Lookup
             // 每次调用都获取一个新的
             public Learn getLearn() {
                 return this.learn;
             }
             public void setLearn(Learn learn) {
                 this.learn = learn;
             }
         }
         ~~~

         3. 如果Learn是使用@Bean注解,则Lookup失效.

   3. 使用ApplicationContextAware接口

      1. ~~~java
         @Component
         public class Write implements ApplicationContextAware {
         
         //    @Autowired
             private Learn learn;
         
             private ApplicationContext applicationContext;
             private String name;
         
             public String getName() {
                 return name;
             }
         
             public void setName(String name) {
                 this.name = name;
             }
         
             //    @Lookup
             // 每次调用都获取一个新的
             public Learn getLearn() {
                 return applicationContext.getBean("learn", Learn.class);
             }
         
             public void setLearn(Learn learn) {
                 this.learn = learn;
             }
         
         
             @Override
             public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
                 this.applicationContext = applicationContext;
             }
         }
         ~~~

6. **FactoryBean和BeanFactory的区别**:FactoryBean本质是一个bean,是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口.对于一些类,实例化过程过于繁琐,xml配置过于复杂,宁愿使用java代码来完成,这时候就需要实现FactoryBean的接口,

   1. ~~~java
      public interface FactoryBean<T> {
          String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";
      
          @Nullable
          T getObject() throws Exception;
      
          @Nullable
          Class<?> getObjectType();
      
          default boolean isSingleton() {
              return true;
          }
      }
      ~~~

   2. getObject()方法就是需要实例化的类.

7. Spring的IOC容器

   1. **容器的启动阶段**

      ​		通过BeanDefinitionReader对加载的Configuration进行解析和分析,并把信息编为BeanDefinition,然后将所有的BeanDefinition注册到相应的BeanDefinitionRegistry中.侧重于对象管理信息的收集.

   2. **Bean的实例化阶段**

      ​		当某个请求通过容器的getBean方法(或隐士的调用)明确的请求某个对象的时候,根据BeanDefinitionRegistry实例化Bean,注入依赖.

8. 容器启动时候的扩展

   - BeanFactoryPostProcess的容器扩展机制.允许容器实例化之前(继在容器启动的最后时刻),对注册到容器BeanDefinition所保存的信息做相应的修改.例如修改Bean定义的某些属性,为Bean定义添加其他的信息等.最常见的例子就是数据库的用户名和密码.
   - PropertyPlaceholderConfigurer,PropertyOverrideConfigurer和CustomEditorConfigure

9. Bean的注册 使用xml通过set方法注入

   1. ~~~xml
      <bean id="clientService" class="com.liuliu.outstanding.factory.ClientService">
          <!-- additional collaborators and configuration for this bean go here -->
      </bean>
      <bean id="son" class="com.liuliu.outstanding.common.Son" parent="father">
              <property name="name" value="Son"></property>
              <property name="clientService" ref="clientService"></property>
      </bean>
      ~~~

   2. 会将clientService注册到properties里面

      1. ![](.\picture\spring揭秘\Bean的注册1.png)

   3. 将Life类通过参数进行注入,参放在了constructorArgumentValues里面

      1. ~~~xml
          <bean id="life" class="com.liuliu.outstanding.common.Life">
             </bean>
             <bean id="son" class="com.liuliu.outstanding.common.Son" parent="father" >
                 <constructor-arg name="life">
                     <ref bean="life"></ref>
                 </constructor-arg>
                 <property name="name" value="Son"></property>
             </bean>
         ~~~

      2. ![](.\picture\spring揭秘\Bean的注册2.png)

10. 反射: **反射就是把java类中的各种成分映射成一个个的Java对象**。

11. Bean的实例化阶段

    1. ![image-20201210162931583](.\picture\spring揭秘\image-20201210162931583.png)
    2. 对于BeanFactory使用的是Bean的懒加载策略,只到A被请求bean或者间接请求,间接是指有依赖到Bean时候.
    3. 虽然是通过BeanDefinition取得实例化信息,通过反射就能创建对象实例,但是并不是直接返回的对象实例,而是BeanWrapper对构造完成的对象实例进行包裹.返回相应的BeanWrapper.进行包裹为的就是第二步设置对象属性.
    4. BeanWarpperImpl实现类作用是对每个Bean实现包裹,设置或者获取Bean的属性,BeanWarpperImpl间接继承了PropertyEditorRegitry,会将注册信息传递给wrapper.

12. BeanPostProcessor

    1. 存在于对象实例化阶段,注意与BeanFactoryPostProcess则是存在于容器的启动阶段.与BeanFactoryPostProcess管理BeanDefinition类似的是,BeanPostProcessor管理的实例化后的对象.
    2. 各种aware接口就是在这postProcessBeforeInitialization处理的.
    3. Spring的AOP功能就是用BeanPostProcess来为对象生成相应的**代理对象**.

13. 自定义BeanPostProcess

   14. 实现BeanPostProcessor接口,重写BeforeInitialization

   15. 将自定义的processor注册到容器.

       - 对于BeanFactory使用configurableBeanFactory.addBeanProcessor方法
       - 对于Application则作为一个Bean配置在xml中就可以了.

- InitializingBean和init-method 对于那些实例化后仍需要更改bean属性的需求.例如计算股市信息去除周六日.InitializingBean是一个接口,init-method则在xml配置有限执行的方法.
- 目前使用**@PostConstruct**和**@PreDestroy**在Bean初始化之前和初始化之后调用.@PostConstruct一般用于集成第三方库,@PreDestroy 则一般用于回调或者自定义对象的销毁.

18. Bean的销毁

    1. BeanFactory容器  调用ConfigurableBeanFactory提供的destroySingletons()的方法.
    2. ApplicationContext容器  调用registerShutdownHook方法.

## 第五章 ApplicationContext容器

1. Resource和ResourceLoader

2. 国际化信息的支持.
   1. 不同的国家配置中进行切换.Locale.US代表美国地区
   
3. MessageSource:统一的国际化访问的方式,传入相应的Locale获取对应的信息.

4. 容器内事件的发布,其实使用的是时间的监听机制.

5. 对配置模块加载的简化.

6. 类初始化后存放的位置**ClassPathXmlApplicationContext** 和 **AnnotationConfigApplicationContext** 分别是通过xml配置的Bean和注解配置的Bean.

7. 当依赖的类相同类型有多个的时候,使用@Primary注解或者**两处**都定义@Qualifier("XXX")确定哪个类优先.

   1. ~~~java
      @Component
      public class GrandFather {
          @Autowired
          @Qualifier("son1")
          private Son son;
      }
      
      @Configuration
      public class LifeConfig {
      
          @Bean
          @Qualifier("son1")
         / @Primary
          public Son getSon1() {
              Son son = new Son();
              son.setName("张三");
              return son;
          }
      
          @Bean
          public Son getSon2() {
              Son son = new Son();
              son.setName("李四");
              return son;
          }
      
      
      }
      ~~~

   2. 


第六章 注解

1. @Autowired :与在xml中定义autowire="bytype",按照类型进行注入,可以定义在方法和属性上面.定义子构造方法上就是通过构造方法注入的,定义在属性上就是通过set方法注入的.
2. 使用@Autowired ,容器会遍历bean,如果符合注入的类的类型,就可以从当前容器管理的对象中获取符合条件的对象,设置给@Autowired所标注的属性域,构造方或者方法的定义.
3. @Quality :和@Autowired搭配使用,可以通过名称进行注入.直接点名从容器中要我们需要的对象.
4. @ Resource :通过名称进行注入.
5. @PostConstruct和@PreDestroy不是服务于依赖注入的,而是生命周期相关的方法,这与Spring的InitializingBean和DisposableBean接口,以及配置项中,init-method和destory-method起到类似的作用.
6. 使用注解其实是类似使用自定义的BeanPostProcess,需要将注解对应的AutowiredAnnotationBeanPostProcessor,必须在配置文件中使用<context:annotation-config>配置开启注解.,开启包扫描的注解<context:component-scan>也注入了这个process.
7. <context:component-scan>这个注解虽然主要的目的是扫描@Controller,@Service,@Dao这类注解,为BeanFactory注入Bean,但是同时也支持<context:annotation-config>这种管理@Autowired依赖的功能.

## 第七章 AOP

1. AOP概念:面向切面编程,它的实现语言为AOL,AOL不是专指某种语言,例如Java的AOL是AspectJ,还有其他语言的AOL,例如C语言的ASpectC.
2. **静态AOP**:即第一代AOP,相应的横切关注点以Aspect形式实现之后,会通过特定的编译器,将实现后的Aspect编译并织入到系统的静态类中.优点是:java虚拟机可以像之前一样加载java类运行.缺点是不够灵活.横切关注点更换位置的时候,就需要更改配置文件的内容.
3. **动态的Aop**,动态Aop是学习的重点,动态Aop的织入过程是在系统运行开始之后进行的,而不是预先编译到系统类中,而且织入信息大都采用外部xml文件格式保存,可以在调整织入点以及织入逻辑地单元的同时,不必变更系统的其它模块.Aspect以class的身份存在于系统中,采用对系统字节码进行操作的方式来完成Aspect到系统的织入.这种方式更容易开发和集成,缺点是需要性能的支持.
4. Spring Aop在无法采用**动态代理机制**进行AOP功能扩展的时候,会使用CGLib库的**动态字节码增强**支持实现AOP的功能扩展.动态代理机制通过反射生成代理对象.动态字节码增强为模块类生成相应的**子类**.
5. java文件执行过程:首先通过javac编译器进行编译生成.class文件,然后类加载器加载.class文件到虚拟机中进行执行.
6. AOP的应用场景:业务需要安全检查和日志记录,如果按照传统面向对象需要在每一个类中添加这两个方法,使用AOP能很好的解决这个问题.
7. Aop的基本概念:
   - 切点(jointPoint):需要对方法增强的地方.可以在异常,方法构造任何地方进行增强的点.
   - pointcut:直接指定jointPoint所在方法名称.与切点的区别是切点是在方法内的某个点,只是描述,与执行无关.pointcut才是定义了方法增强的位置.
   - advice:通知 是单一横切关注点的载体,代表会织入到jointPoint的横切逻辑.相当于执行的方法体.
     - beforeAdvice
     - afterAdvice
     - aroundAdvice
     - **introduction**(非常特殊)
8. ProxyFactory类则是Spring AOP中最通用的织入器.

## 第八章 Spring AOP概述及其实现机制

1. **spring aop的实现机制**:Sring Aop属于第二代AOP,采用**动态代理**和**字节码生成**的技术实现,与最初的AspectJ采用编译器将横切逻辑织入目标对象不同,动态代理机制和字节码生成都是在运行期间为目标对象生成一个代理对象,而将横切逻辑织入到这个代理对象中,系统最终使用的是织入了横切逻辑的代理对象,而不是真正的目标对象.
2. 静态代理是将代码写入到代理中,但是如果代理的类型发生改变,即使增强的方法是相同的但依然需要实现一遍,所以需要使用动态代理的方式.
3. 既然只要生成的代理具有相同的行为,那么就不会仅仅有jdk动态代理通过实现接口这一种的方式了,遵循里氏替换原则的cgLib动态代理,通过继承的方式也能实现动态的代理.cglib只是提供了另外的一种方式,它是使用继承的方式进行的. **通过接口使用的动态代理技术,通过CGlib使用的是字节码生成的技术.**
4. cglib动态代理:当某一类没有实现任何的接口时候使用.
5. 静态代理每遇见一个类就需要生成一个代理,所以一般使用动态的代理,主要由一个类和一个接口组成,Proxy类和InvocationHandler接口.
6. 动态字节码生成:实现MethodInterceptor接口定义增强方法,使用Enhancer设置需要增强的对象,调用Create方法.

## 第九章 Spring AOP 一世

1. Spring Aop的joinpoint是面向方法的,如果想面向属性,则需要借助AspectJ.

2. PointCut中有两个方法,匹配类型的匹配和方法的匹配.

   1. ```java
      public interface Pointcut {    
          Pointcut TRUE = TruePointcut.INSTANCE;    
          ClassFilter getClassFilter();   
          MethodMatcher getMethodMatcher();}
      ```

3. ```java
   public interface ClassFilter {
       ClassFilter TRUE = TrueClassFilter.INSTANCE;
   
       boolean matches(Class<?> var1);
   }
   ```

   ClassFilter比较简单,就是判断类型是否匹配,如果匹配了就返回true;

4. ~~~java
   public interface MethodMatcher {
       MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
   
       boolean matches(Method var1, Class<?> var2);
   
       boolean isRuntime();
   
       boolean matches(Method var1, Class<?> var2, Object... var3);
   }
   ~~~

   matches方法提供了关注参数过滤和不关注参数过滤的两种方法.isRuntime为返回false时候,表示不会考虑joinpoint的**方法参数**,称为StaticMethodMatcher,反之为DynamicMethodMatcher.

   spring最主要的支持就是方法的拦截.

5. pointcut族谱

   ![image-20201226173753955](.\picture\spring揭秘\image-20201226173753955.png)

6. 常见的pointcut

   1. ![image-20201226173941374](.\picture\spring揭秘\image-20201226173941374.png)

7. 可以通过交集或者并集的方式组合pointcut.

8. 常见的PonitCut

   - NameMatchMethodPointCut
     - 属于**StaticMethodMatcherPointcut**的子类,可以根据自身指定的方法名和Jointpoint处的方法名称进行匹配.但是无法对重载的方法进行匹配.

9. pointcut在Spring中可以作为一个普通的Bean,配置在xml中.

10. 前置通知可以用来检测文件的位置是否存在.

11. 异常后置通知可以第一时间知道报出异常.

12. AfterReturnAdvice:只有方法正常返回的情况下,才会被执行,并且它不能更改返回值,所以不适合在方法中定义清理资源的这类功能,典型的使用地方是当数据处理完成后,会向数据库添加成功的信息,可以做这种场景的使用.

13. per-class类型的Advice,即会在目标类所有对象实例之间共享.这种类型的Advice通常只提供方法的拦截,不会为类保存状态 或者添加新的属性.per-instance:spring Aop中Introduction是唯一一种.不改动目标类定义的情况下,为目标类添加新的属性和行为.

14. ![image-20201219115741835](.\picture\spring揭秘\image-20201219115741835.png)

15. ![image-20201219115529841](.\picture\spring揭秘\image-20201219115529841.png)

16. Aspect在Spring中是Advisor,作用是将Pointcut和Advice结合.

17. 通过methodInceptor的invoke方法的methodInvocation参数,我们可以控制对相应joinpoint的拦截行为.通过调用methodInvocation的process方法可以使程序继续的执行.

18. IntroductionAdvisor 只能用于类级别的拦截.

19. 把pointcut和advisor连接起来的的类是PointcutAdvisor家族类,常用的是DefaultPointcutAdvisor类.

20. 当有多个横切逻辑的时候,需要指定他们的优先级,如果没有指定,则按照配置文件声明的顺序,谁先在前谁先执行.多个横切逻辑有时候会因为顺序发生异常.

21. ProxyFactory是一个织入器.

    1. ~~~java
       public ProxyFactory(Object target) {
       		setTarget(target);
           // setInterfaces 设置增强的接口.
       		setInterfaces(ClassUtils.getAllInterfaces(target));
       	}
       ~~~

    2. 代理可以强制转换成接口的类型,但是不能转换成接口的实现类.

    3. proxyFactory.setProxyTargetClass(true); 则使用CGLib基于类的中动态代理模式.

    4. ![](.\picture\spring揭秘\AopProxy代理.png)

22. JDK动态代理是通过Proxy.newProxyInstance()方法声场代理类, CGLib动态代理使用Enhancer enhancer = createEnhancer(); 设置父类和接口,最终调用enhancer.create()方法生成代理类.

    1. ![](.\picture\spring揭秘\JDK动态代理.png)

23. AopProxyFactory 工厂生成代理类,只有一个实现类DefaultAopProxyFactory

    1. ~~~java
       public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {
       
       	private static final long serialVersionUID = 7930414337282325166L;
       
       
       	@Override
       	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
               // 如果 config.isOptimize() 或者  config.isProxyTargetClass() 返回true 或者是没有接口的实现
               // 则使用CGLib动态代理.
       		if (!NativeDetector.inNativeImage() &&
       				(config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config))) {
       			Class<?> targetClass = config.getTargetClass();
       			if (targetClass == null) {
       				throw new AopConfigException("TargetSource cannot determine target class: " +
       						"Either an interface or a target is required for proxy creation.");
       			}
       			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass) || ClassUtils.isLambdaClass(targetClass)) {
       				return new JdkDynamicAopProxy(config);
       			}
       			return new ObjenesisCglibAopProxy(config);
       		}
       		else {
       			return new JdkDynamicAopProxy(config);
       		}
       	}
       
       	/**
       	 * Determine whether the supplied {@link AdvisedSupport} has only the
       	 * {@link org.springframework.aop.SpringProxy} interface specified
       	 * (or no proxy interfaces specified at all).
       	 */
       	private boolean hasNoUserSuppliedProxyInterfaces(AdvisedSupport config) {
       		Class<?>[] ifcs = config.getProxiedInterfaces();
       		return (ifcs.length == 0 || (ifcs.length == 1 && SpringProxy.class.isAssignableFrom(ifcs[0])));
       	}
       
       }
       ~~~

    2. DefaultAopProxyFactory能正常的工作,需要依赖于ProxyConfig和Advice,他们的继承关系如下.

       1. ![](.\picture\spring揭秘\ProxyFactory.png)

24. AdvisedSupport是生成代理对象所需要的信息的载体.一类是ProxyConfig为统领,承载生成代理对象的控制信息,另一类以Advised为旗帜,承载生成代理对象所需要的信息,如advice,advisor等.

25. 通过jdk动态代理生成的代理类,需要强转为实现的接口,因为只有接口才有和横切逻辑,当使用Cglib的方式,代理的对象就要转成实体类,因为他是通过类进行增强的.

26. ProxyFactoryBean容器中的织入器.BeanFactory 和FactoryBean的区别?

    1. 如果容器中某个对象持有FactoryBean的引用,它取得的不是FactoryBean本身,而是getObject()方法返回的对象.

    2. 所以引用ProxyFactoryBean返回的是某个Proxy对象,等价于Proxy+FactoryBean.

    3. ~~~java
       /**
       	 * Return a proxy. Invoked when clients obtain beans from this factory bean.
       	 * Create an instance of the AOP proxy to be returned by this factory.
       	 * The instance will be cached for a singleton, and create on each call to
       	 * {@code getObject()} for a proxy.
       	 * @return a fresh AOP proxy reflecting the current state of this factory
       	 */
       	@Override
       	@Nullable
       	public Object getObject() throws BeansException {
       		initializeAdvisorChain();
       		if (isSingleton()) {
       			return getSingletonInstance();
       		}
       		else {
       			if (this.targetName == null) {
       				logger.info("Using non-singleton proxies with singleton targets is often undesirable. " +
       						"Enable prototype proxies by setting the 'targetName' property.");
       			}
       			return newPrototypeInstance();
       		}
       	}
       ~~~

    4. 基于ProxyFactoryBean的配置

       1. ![](.\picture\spring揭秘\ProxyFactoryBean配置.png)
       2. 注意MockTask 是代理对象而不是目标对象,否则不会增强.

27. SpringAOP二代其实底层使用的还是一代.

28. 注解的方式advice的顺序是,谁先声明谁先执行,但是后置advice,谁先声明,谁就是最后执行优先.

29. 让spring扫描切面的注解,在配置文件中配置

    1. ```xml
       <aop:aspectj-autoproxy proxy-target-class="true">
       ```

    2. 使用上面的配置,可以达到基于DTD的配置方式中,直接声明AnnotationAwareAspectJAutoProxyCreator相同的效果.

## 第十章 Spring AOP 二世

1. Spring AOP 只支持方法级别的Joinpoint.

2. @Aspect形式Pointcut表达式的标志符

   1. execution:方法执行的时候进行拦截,需要定义方法的返回类型,方法名以及参数
   2. within:标志符只接受类型声明,匹配指定类的所有方法的执行.
   3. this,target:在AspectJ中,this指代调用方法一方所在的对象,target指代被调用方法所在的对象
   4. args:捕捉指定参数类型,而不管该方法在什么类型中被声明.

3. .**在Spring AOP中,this指代的是目标对象的代理对象,而target如其名指的是目标对象.**

4. execute和within都能定义Pointcut的jointpoint.

5. 当Advice声明在不同的Aspect内的时候,需要实现Ordered接口,通过getOrder()方法来判断,越小优先级越高.否则,advice的执行顺序是不确定的.

6. 通知的优先级

   1. 可以通过@Order定义@Aspect顺序.
   2. Spring Framework 5.2.7之后相同条件先通知的执行的顺序是:@Around`, `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing.

7. 通过自动代理机制处理横切逻辑到目标对象的织入,是28条说明的样子,但是当通过编程的方式处理时,advice由添加到AspectJProxyFactory的顺序来决定的.

8. 最新的xml配置都是基于schema的格式.

9. ```xml
   <apo:config proxy-target-class="false">
   	<aop:pointcut></aop:pointcut>    
       <aop:advisor></aop:advisor>
       <aop:aspect></aop:aspect>
   </aop:config>
   ```

   pointcout,advisor,aspect顺序必须是这样.

10. Around Advice必须要结合ProceedingjoniPoint类型进行使用.

11. **内部方法调用失败的问题:**当目标对象中原始方法调用依赖其它对象,那没问题,可以为目标对象注入所依赖的对象的代理,但是当目标对象中的原始方法调用直接调用自身方法的时候,也就是说,它依赖于自身所定义的其它方法的时候,就会出现找不到依赖方法的异常.因为method1调用的是TargetObject对象上的method2,而不是ProxyObject上的method2增强过的方法.

    1. 解决:在调用自身的方法的时候,使用AOPContext.curretnProxy()调用,并设置weaver.setExposeProxy(true);

12. AOP经常使用的场景:异常处理,安全检查,缓存.

---

## 第十三章 统一的数据访问异常层次体系

1. 当需要连接不同的数据源的时候,使用需要变换的就是连接数据源的代码,首先面对的问题是连接时候抛出的受检异常例如SqlException.解决办法
   - 封装成运行时异常.
   - 每种类型进行分类,提示用户.

## 第十四章 JDBC API的最佳实践

1. 为了实现数据库差异性以及事物管理,spring一般使用封装过的datasource.

---

## 第十七章 事务管理

1. 事物的管理主要做了三件事情:
   1. 将事务资源进行统一管理.
   2. 事务异常转义.
   3. 事务开始和结束的定义.
2. Spring的事物框架理论设计原则:让事物管理的关注点和数据访问关注点相分离.
3. 事物的实现思路:例如JDBC的局部事物控制是由同一个Connection来完成,所以要保证数据访问方法处于一个事物中,我们就要保证连接的是同一个Connection,为了不让代码与Connection耦合,将Connection放在统一的地方,绑定到当前的线程,谁需要谁获取.
4. DatasourceUtils最主要的功能是对Connection的管理.会从PlatformTransactionManager中获取connection,如果当前线程没有绑定connection,通过数据访问对象的Datasource中获取新的connection,否则使用绑定的connection,用来保证数据访问处于同一个事物中.
5. Spring的事务抽象包括三个主要的接口:
   1. PlatformTransactionManager:负责界定事务边界.
   2. TransactionDefinition:负责定义事务相关属性,包括隔离级别,传播行为等.PlatformTransactionManager根据相关TransactionDefinition属性开启相关事务.
      1. 隔离级别,事物传播,超时时间和是否为只读的事物.
   3. TransactionStatus:事务开启之后到事务结束期间的状态由Status维护.
6. **事物的传播行为:**
   - Required:如果当前存在一个事物,则加入当前的事物,如果没有就创建,总之,至少要保证在一个事物中运行,是默认的事物传播行为.
   - Support:如果当前存在一个事物就加入,不存在则直接执行.对查询方法是比较好的选择.
   - Mandatory:强制要求当前存在一个事务,如果不存在就抛出异常.需要事务但是自身不管理事务的提交和回滚.
   - Required_New:不过当前是否存在事务,都会创建新的事务,如果存在事务,就将它挂起.适用于某个业务对象所做的事情,不想影响外层事务,例如日志信息的更新失败.
   - Not_Supported:不支持当前事务,而是没有事务的情况下直接执行,如果当前存在事物的话,原则上被挂起.
   - Never;不需要事务,存在事务就抛出异常.
   - Nested:如果存在事务,则在当前事务的一个嵌套事务中执行,如果没有,创建新的事物,但是他创建的事务不同于Required_New,他比外层的事务优先级有低,而Required_New创建的事务是与外层事务等级相同的,创建的时候外层事务是挂起的状态.适用于将一个大事务分解成多个小事务处理的场景.
7. TransactionStatus可以通过SavePoint创建嵌套事物,SavePoint可以作为一个临时状态点,方便子事物的回滚.
8. 介绍了Spring各种时代事务的配置方式.现在常用的是@Transaction注解,在里面可以定义事务的传播行为和事务的隔离度,还可以通过xml用<tx></tx>进行配置.
   1. ![image-20210103125257642](.\picture\spring揭秘\image-20210103125257642.png)
9. ThreadLocal:不需要管理多个线程共享,但是需要多个线程进行传递的资源,例如JDBC连接Connection,通过TheadLocal为每个线程分配一个他们各自持有的Connection,将资源绑定到线程上.避免多个线程同时访问出现混乱.它与线程绑定.ThreadLocal是当前线程的分身,ThreadLocal工具类底层就是一个相当于一个Map,key存放的当前线程,value存放需要共享的数据.
10. 分布式事务,涉及到ResourceManager,XAResource,TransactionManager三者直接的交互实现分布式事务.

---

## 第二十二章 Spring的Web MVC 框架

1. JSP:将Servlet中的视图渲染逻辑以独立的单元抽取出来.
2. 控制器,模型,视图(MVC)三者配合实现数据展示.控制器需要根据配置文件找到请求与页面的配置关系.
2. ContextLoaderListener的职责是将整个的Web应用程序加载顶层的WebApplicationContext.WebApplication主要用于提供所有的中间层服务.所使用的DataSource,Dao,Service等都在WebApplication中注册.
3. ![image-20210103221028161](.\picture\spring揭秘\image-20210103221028161.png)

   - HandlerMapping:定义了每个请求对应访问的Controller.
   - ViewResolver和View:将视图内容按照字节和字符分为两种类型.View来处理视图内容的具体工作,而ViewResolver类似于HandleMapping功能,记录了视图的对应的关系.
4. 通过一个例子演示了如何配置.

## 第二十四章 近距离接触Spring MVC主要角色

1. ![image-20210106162538865](.\picture\spring揭秘\image-20210106162538865.png)
2. HandlerMapping主要是根据配置,为url找到对应的Controller类,SpringMVC的web应用中,我们为DispatchServlet提供多个Handler,按照优先级进行匹配,如果HandlerMapping能够返回不可用的Handler,继续向下查找.知道找到可用的Handler.
3. ViewResolver作用为DispatchServlet返回一个可以的view.
4. HandleMapping不仅仅是返回Controller这这一种类型,为了能够找到对应的Handler来处理,使用了HandlerAdapter来找到他们的关系.
5. HandlerAdaptor的主要工作知识调用这个HandlerAdaptor"认识"的Handler的web请求处理方法,然后将处理结果转换为DisaptcherServlet统一使用的ModelAndView.其实就是适配器模式.
6.   HandlerInterceptor在可以设置在页面渲染之前执行的方法和渲染之后执行的方法,它的父类其实HandleMapping.配置xml的时候也是配置在HandlerInterceptor之下的.

---

## 第三十一章 Spring中任务的调度和线程池的支持

1. Quart:任务调度框架
2. SimpleAsyncTaskExecutor:提供最基础的异步执行能力,而实现的方式则是为每个任务创建新的线程.可以设置线程数的上限.
3. ThreadPoolTaskExecutor:为每一个任务创建线程非常的耗费资源.所以需要使用线程池来管理.允许对线程池中的参数做一些设置.
4. 其实这一章主要讲的就是定时任务调度.
5. Spring Remoting:因为想要远程调用服务,会涉及到很多交互细节,不能将他们全都交给业务对象(服务端或者客户端),所以需要将此业务进行剥离,产生了Spring Remoting这种对象.

## 常见问题

1. 循环依赖

   1. Spring怎么解决循环依赖?

      1. 使用三级缓存:
         1. 一级缓存:singletonObjects:保存了完成初始化的对象
         2. 二级缓存earlySingletonObjects:保存了出现了循环依赖,并且需要提前给其它类用的类(被AOP).而且保证了每次生成的代理对象都是同一个.
         3. 三级:singleFactories:提前暴露未初始化完成的对象,避免依赖循环.添加了一个lambda表达式,调用三级缓存,执行lambda表达式,会判断是否被切(AOP),如果被切,就生成代理对象,如果没有就返回原对象.将返回的对象放入到二级缓存中去.

   2. 为什么使用三级缓存,使用二级缓存是否可以?

      ​	如果没有AOP使用一个二级缓存就可以,如果为二级缓存是为了当类没有初始化完成,提前进行了AOP,保证拿到的是同一个代理对象.

   3. 为什么不能解决构造方法和多例的循环依赖?

      ​	通过缓存直接暴露,而多例每次都创建一个新的对象,构造方法直接的依赖实例Bean,都没有提前暴露的条件.

      ​	使用@lazy进行注释,当出现依赖实际是创建了一个代理类,A依赖B1(代理类) , B依赖A 就不会出现循环依赖了.

2. Bean的生命周期![](.\picture\spring揭秘\Bean的生命周期.png)

   1. 实例化(Instantiation)

      1.   对于[BeanFactory](https://so.csdn.net/so/search?q=BeanFactory&spm=1001.2101.3001.7020)容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一尚未初始化的依赖时，容器会调用createBean进行实例化。

      2. 对于ApplicationContext容器，当容器启动结束后，通过获取[BeanDefinition](https://so.csdn.net/so/search?q=BeanDefinition&spm=1001.2101.3001.7020)对象中的信息，实例化所有的bean。

   2. 属性设置(populate)

      1.  实例化后的对象被封装在BeanWrapper对象中，Spring根据BeanDefinition中的信息以及通过BeanWrapper提供的设置属性的接口完成**属性设置与依赖注入**。主要是依赖属性的处理,例如实例化A,发现A依赖B,怎会先处理B,将B创建好后进行属性的设置.

   3. **BeanPostProcessor前置处理**

      1. 如果想对Bean进行一些自定义的前置处理，那么可以让Bean实现了BeanPostProcessor接口，那么将会调用postProcessBeforeInitializ（Object obj，String s）方法。

   4. **InitialzingBean**

      ​    如果Bean实现了InitialzingBean接口，执行afterPropertiesSet()方法。

   5. **init-method**

      ​    如果Bean在Spring配置文件中配置了init-method属性，则会自动调用其配置的初始化方法。

   6. **BeanPostProcessor后置处理**

      ​    以上几个步骤完成后，Bean已经正确创建。

      ​    如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitiazation（Object obj，String s）方法，由于这个方法是在Bean初始化结束时调用，所以可以被应用于内存或缓存技术；

   7. **DisposableBean**

      ​    当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy（）方法；

   8. **destory-method**

      ​    最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

      