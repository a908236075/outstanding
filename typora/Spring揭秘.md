# Spring揭秘

### 第一章Spring框架总体结构

1. IOC容器,AOP, DAO JDBC和事物管理,ORM Mybatis,SpringMVC.
2. ![image-20201209100946514](.\picture\spring揭秘\image-20201209100946514.png)

### 第二章 IOC的基本概念

1. IOC:控制反转,由IOC Service Provider统一管理,将被依赖的对象注入到注入对象中.

2. Bean注入方式

   - 构造方法注入:

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

   - Set方法注入

     - ![image-20201209134553435](.\picture\spring揭秘\image-20201209134553435.png)

     - 应为set方法可以命名,会比构造方法好一些,setter方法可以别继承,可以设置默认值,缺点是无法再构造完成后马上进入就绪的状态.

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

3. 对IOC真正的理解

   1. spring是单例模式的,如果没有IOC,每一个都要编写单例的方法.
   2. 对于Bean的**依赖管理**,如果类A依赖B,C,D类,不需要一个一个的new这样耦合在代码的方式,通过IOC可以实现Bean的管理和解耦.

4. Autowired注解相当于在xml中写了一个Bean注解,Autowired注入成员变量(写在类名上)，利用set方法注入，要等类加载完了才注入bean.

   Autowired注入构造方法中，利用构造器注入，有先后依赖关系.

   setter方法注入，setter代码冗长，不能将属性设置为final。

5. IOC Service Provider

   - 业务对象的构建管理
   - 业务对象之间的依赖绑定
     - 直接编码的方式:在容器中直接用代码管理关系.所有方式的最终方式.
       - ![image-20201209145322335](.\picture\spring揭秘\image-20201209145322335.png)
     - 配置文件的方式,xml的方式最常见.
     - 注解的方式,其实注最终还是编码的方式来确定注入的关系.

### 第四章 Spring的IOC容器之BeanFactory

1. spring Ioc容器和Ioc provider的关系
   
   ![image-20201209151641139](.\picture\spring揭秘\image-20201209151641139.png)
   
2. spring提供了两种容器类型:BeanFactory和ApplicationContext

   - BeanFactory:基础类型IOC容器,提供完整的Ioc服务支持.默认采用延迟初始化策略(懒加载).启动时间快,需要的系统资源少.
   - ApplicationContext:在BeanFactory上构建,;对其进行了功能上的升级,比如事件发布,国际化信息支持等.默认启动时完成所有初始化,需要更多的系统资源.
   - ![image-20201209152151032](.\picture\spring揭秘\image-20201209152151032.png)

3. BeanFactory,BeanDefinitionRegister以及DefaultListtableBeanFactory的关系.

   1. BeanFactory 只是一个接口，我们最终需要一个该接口的实现来进行实际的Bean的管理， Default-
      ListableBeanFactory 就是这么一个比较通用的 BeanFactory 实现类。 DefaultListableBean-
      Factory 除了间接地实现了 BeanFactory 接口，还实现了 BeanDefinitionRegistry 接口，该接口才
      是在 BeanFactory 的实现中担当Bean注册管理的角色。基本上， BeanFactory 接口只定义如何访问容
      器内管理的Bean的方法，各个 BeanFactory 的具体实现类负责具体Bean的注册以及管理工作。
      BeanDefinitionRegistry 接口定义抽象了Bean的注册逻辑。通常情况下，具体的 BeanFactory 实现
      类会实现这个接口来管理Bean的注册。它们之间的关系如图4-3所示

   ![image-20201209160039061](.\picture\spring揭秘\image-20201209160039061.png)

4. BeanFactory的对象注册与依赖绑定方式

   - 直接编码的方式:BeanDefinitionRegistry接口定义Bean的注册逻辑.每一个受管理的对象在容器中都会有一个BeanDefinition(instance)的实例与之相对应.
   - 外部配置文件方式
     - 一般用BeanDefinitionReader来读取配置文件.
     - properties
     - xml    **spring读取xml配置文件的过程?**
       -  set方法,构造函数方法,
       - list,set,map 复杂数据结构的封装.各种标签的介绍.
       - byName和byType自动绑定,不用我们手写class类,省去了一些代码量,但是类型唯一时候才能保证成功.
       - lazy-init 主要针对application容器设置延时初始化.
   - 注解方式.注解注入其实是使用注解的方式进行构造器注入或者set注入。使用哪种方式注入,通过位置来判断,如果@autowired在构造方法上就是使用了构造器注入的.**这里需要区分Bean注入的方式和Bean注册方式的区别.**注解的方式其实是Bean注册方式的一种.注册是管理的类之前的关系,注入关注的是类的生成.
     - Bean的scope
       - 单例 与IOC容器拥有相同的寿命.
       - 多例  容器为每次请求创建一个对象 ,随后不在管理他的生命周期.
       - request,session,global session:request是多例的一个特例,只是拥有了具体的使用场景.
     - FactoryBean本质上一个Bean,只不过这种类型的bean本身,就是生产对象的工厂.不要与BeanFactory相混淆.

5. Bean的Scope:配置中的Bean可以看多是模板,容器会根据他们构造对象,的说那是要创建多少个,构造完成后对象实例存活多久,则是由容器根据Bean的scope语义来决定的.

6. 注意:GOF的单例模式和spring的Singleton模式是不同的,标记为Singleton的bean是由容器来保证这类的bean在同一个容器中只存在一个共享实例;而Singleton模式则是保证同一个ClassLoader中只存在一个这种类型的实例.而设计模式中，我们是对构造方法私有化，进行单例模式，用户从而不能new多个实例。

7. 单例:容器中只存在一个共享的实例.存活时间:从容器启动,到它第一次被请求实例化开始,只要是容器不销毁或者退出,改Bean就会一直存活.多例:每次请求都会创建一个全新的对象,只负责创建,后期的对象的生命周期由请求方来维护.

8. request,session,global session只适用于web应用程序.

9. 小心prototype(多例的陷阱),有时候即使我们配置了多例,但是程序取对象的时候只取同一个对象,这是因为引用不当,每次请求没有向IOC的容器中获取对象,解决方法如下:

   1. 需要配置lookup-method方法注入的方式.

   ```xml
   <bean id="persister" class="">
   	<lookup-method name  bean></lookup-method>
   </bean>
   ```

   2. 使用BeanFactoryAware接口
   3. 方法替换.**???**

10. **FactoryBean和BeanFactory的区别**:FactoryBean本质是一个bean,是Spring容器提供的一种可以扩展容器对象实例化逻辑的接口.

11. Spring的IOC容器

    1. **容器的启动阶段**

       ​		通过BeanDefinitionReader对加载的Configuration进行解析和分析,并把信息编为BeanDefinition,然后将所有的BeanDefinition注册到相应的BeanDefinitionRegistry中.侧重于对象管理信息的收集.

    2. **Bean的实例化阶段**

       ​		当某个请求通过容器的getBean方法(或隐士的调用)明确的请求某个对象的时候,根据BeanDefinitionRegistry实例化Bean,注入依赖.

12. 容器启动时候的扩展

    - BeanFactoryPostProcess的容器扩展机制.允许容器实例化之前(继在容器启动的最后时刻),对注册到容器BeanDefinition所保存的信息做相应的修改.例如修改Bean定义的某些属性,为Bean定义添加其他的信息等.最常见的例子就是数据库的用户名和密码.
    - PropertyPlaceholderConfigurer,PropertyOverrideConfigurer和CustomEditorConfigure

13. Bean的实例化阶段

    1. ![image-20201210162931583](.\picture\spring揭秘\image-20201210162931583.png)
    2. 对于BeanFactory使用的是Bean的懒加载策略,只到A被请求bean或者间接请求,间接是指有依赖到Bean时候.
    3. 虽然是通过BeanDefinition取得实例化信息,通过反射就能创建对象实例,但是并不是直接返回的对象实例,而是BeanWrapper对构造完成的对象实例进行包裹.返回相应的BeanWrapper.进行包裹为的就是第二步设置对象属性.
    4. BeanWarpperImpl实现类作用是对每个Bean实现包裹,设置或者获取Bean的属性,BeanWarpperImpl间接继承了PropertyEditorRegitry,会将注册信息传递给wrapper.

14. BeanPostProcessor

    1. 存在于对象实例化阶段,注意与BeanFactorPostProcess则是存在于容器的启动阶段.与BeanFactoryPostProcess管理BeanDefinition类似的是,BeanPostProcessor管理的实例化后的对象.
    2. 各种aware接口就是在这postProcessBeforeInitialization处理的.
    3. Spring的AOP功能就是用BeanPostProcess来为对象生成相应的**代理对象**.

15. 自定义BeanPostProcess

   16. 实现BeanPostProcessor接口,重写BeforeInitialization

   17. 将自定义的processor注册到容器.
       - 对于BeanFactory使用configurableBeanFactory.addBeanProcessor方法
       - 对于Application则作为一个Bean配置在xml中就可以了.
- InitializingBean和init-method 对于那些实例化后仍需要更改bean属性的需求.例如计算股市信息去除周六日.InitializingBean是一个接口,init-method则在xml配置有限执行的方法.
       
18. Bean的销毁

    1. BeanFactory容器  调用ConfigurableBeanFactory提供的destroySingletons()的方法.
    2. ApplicationContext容器  调用registerShutdownHook方法.

### 第五章 ApplicationContext容器

1. Resource和ResourceLoader
2. 国际化信息的支持.
   1. 不同的国家配置中进行切换.Locale.US代表美国地区
3. MessageSource:统一的国际化访问的方式,传入相应的Locale获取对应的信息.
4. 容器内事件的发布,其实使用的是时间的监听机制.
5. 对配置模块加载的简化.

### 第六章 注解

1. @Autowired :与在xml中定义autowire="bytype",按照类型进行注入,可以定义在方法和属性上面.定义子构造方法上就是通过构造方法注入的,定义在属性上就是通过set方法注入的.
2. 使用@Autowired ,容器会遍历bean,如果符合注入的类的类型,就可以从当前容器管理的对象中获取符合条件的对象,设置给@Autowired所标注的属性域,构造方或者方法的定义.
3.  @Quality :和@Autowired搭配使用,可以通过名称进行注入.直接点名从容器中要我们需要的对象.
4. @ Resource :通过名称进行注入.
5. @PostConstruct和@PreDestroy不是服务于依赖注入的,而是生命周期相关的方法,这与Spring的InitializingBean和DisposableBean接口,以及配置项中,init-method和destory-method起到类似的作用.
6. 使用注解其实是类似使用自定义的BeanPostProcess,需要将注解对应的AutowiredAnnotationBeanPostProcessor,必须在配置文件中使用<context:annotation-config>配置开启注解.,开启包扫描的注解<context:component-scan>也注入了这个process.
7. <context:component-scan>这个注解虽然主要的目的是扫描@Controller,@Service,@Dao这类注解,为BeanFactory注入Bean,但是同时也支持<context:annotation-config>这种管理@Autowired依赖的功能.

### 第七章 AOP

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

### 第八章 Spring AOP概述及其实现机制

1. **spring aop的实现机制**:Sring Aop属于第二代AOP,采用**动态代理**和**字节码生成**的技术实现,与最初的AspectJ采用编译器将横切逻辑织入目标对象不同,动态代理机制和字节码生成都是在运行期间为目标对象生成一个代理对象,而将横切逻辑织入到这个代理对象中,系统最终使用的是织入了横切逻辑的代理对象,而不是真正的目标对象.
2. 静态代理是将代码写入到代理中,但是如果代理的类型发生改变,即使增强的方法是相同的但依然需要实现一遍,所以需要使用动态代理的方式.
3. 既然只要生成的代理具有相同的行为,那么就不会仅仅有jdk动态代理通过实现接口这一种的方式了,遵循里氏替换原则的cgLib动态代理,通过继承的方式也能实现动态的代理.cglib只是提供了另外的一种方式,它是使用继承的方式进行的. **通过接口使用的动态代理技术,通过CGlib使用的是字节码生成的技术.**
4. cglib动态代理:当某一类没有实现任何的接口时候使用.
5. 静态代理每遇见一个类就需要生成一个代理,所以一般使用动态的代理,主要由一个类和一个接口组成,Proxy类和InvocationHandler接口.
6. 动态字节码生成:实现MethodInterceptor接口定义增强方法,使用Enhancer设置需要增强的对象,调用Create方法.

### 第九章 Spring AOP 一世

1. Spring Aop的joinpoint是面向方法的,如果想面向属性,则需要借助AspectJ.

3. PointCut中有两个方法,匹配类型的匹配和方法的匹配.

   1. ```java
      public interface Pointcut {    
          Pointcut TRUE = TruePointcut.INSTANCE;    
          ClassFilter getClassFilter();   
          MethodMatcher getMethodMatcher();}
      ```

4. ```java
   public interface ClassFilter {
       ClassFilter TRUE = TrueClassFilter.INSTANCE;
   
       boolean matches(Class<?> var1);
   }
   ```

   ClassFilter比较简单,就是判断类型是否匹配,如果匹配了就返回true;

5. ~~~java
   public interface MethodMatcher {
       MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;
   
       boolean matches(Method var1, Class<?> var2);
   
       boolean isRuntime();
   
       boolean matches(Method var1, Class<?> var2, Object... var3);
   }
   ~~~

   matches方法提供了关注参数过滤和不关注参数过滤的两种方法.isRuntime为返回false时候,表示不会考虑joinpoint的方法参数,称为StaticMethodMatcher,反之为DynamicMethodMatcher.

   spring最主要的支持就是方法的拦截.

5. pointcut族谱

   ![1608558348213](C:\Users\liubt\AppData\Roaming\Typora\typora-user-images\1608558348213.png)

6. 常见的pointcut

   1. 

7. 可以通过交集或者并集的方式组合pointcut.

8. 常见的PonitCut
   - NameMatchMethodPointCut
     - 属于StaticMethodMatcherPointcut的子类,可以根据自身指定的方法名和Jointpoint处的方法名称进行匹配.但是无法对重载的方法进行匹配.

9. pointcut在Spring中可以作为一个普通的Bean,配置在xml中.

10. 前置通知可以用来检测文件的位置是否存在.

11. 异常后置通知可以第一时间知道报出异常.

12. AfterReturnAdvice:只有方法正常返回的情况下,才会被执行,并且它不能更改返回值,所以不适合在方法中定义清理资源的这类功能,典型的使用地方是当数据处理完成后,会向数据库添加成功的信息,可以做这种场景的使用.

13. per-class类型的Advice,即会在目标类所有对象实例之间共享.per-instance:spring Aop中Introduction是唯一一种.不改动目标类定义的情况下,为目标类添加新的属性和行为.

14. ![image-20201219115741835](.\picture\spring揭秘\image-20201219115741835.png)

15. ![image-20201219115529841](.\picture\spring揭秘\image-20201219115529841.png)

16. IntroductionAdvisor 只能用于类级别的拦截.

17. 当有多个横切逻辑的时候,需要指定他们的优先级,如果没有指定,则按照配置文件声明的顺序,谁先在前谁先执行.多个横切逻辑有时候会因为顺序发生异常.

18. ProxyFactory是一个织入器.ProxyFactoryBean容器中的织入器.

19. SpringAOP二代其实底层使用的还是一代.

20. 注解的方式advice的顺序是,谁先声明谁先执行,但是后置advice,谁先声明,谁就是最后执行优先.

21. 测试提交

