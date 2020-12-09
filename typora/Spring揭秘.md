# Spring揭秘

### 第一章Spring框架总体结构

1. IOC容器,AOP, DAO JDBC和事物管理,ORM Mybatis,SpringMVC.
2. ![image-20201209100946514](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20201209100946514.png)

### 第二章 IOC的基本概念

1. IOC:控制反转,由IOC Service Provider统一管理,将被依赖的对象注入到注入对象中.

2. Bean注入方式

   - 构造方法注入:

     - ![image-20201209134452067](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20201209134452067.png)
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

     - ![image-20201209134553435](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20201209134553435.png)

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
   2. 对于Bean的依赖管理,如果类A依赖B,C,D类,不需要一个一个的new这样耦合在代码的方式,通过IOC可以实现Bean的管理和解耦.

4. Autowired注解相当于在xml中写了一个Bean注解,Autowired注入成员变量(写在类名上)，利用field反射注入，要等类加载完了才注入bean.

   Autowired注入构造方法中，利用构造器注入，有先后依赖关系.

   setter方法注入，setter代码冗长，不能将属性设置为final。

5. IOC Service Provider

   - 业务对象的构建管理
   - 业务对象之间的依赖绑定
     - 直接编码的方式:在容器中直接用代码管理关系.所有方式的最终方式.
       - ![image-20201209145322335](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20201209145322335.png)
     - 配置文件的方式,xml的方式最常见.
     - 注解的方式,其实注最终还是编码的方式来确定注入的关系.

### 第四章 Spring的IOC容器之BeanFactory

1. spring Ioc容器和Ioc provider的关系
   
   ![image-20201209151641139](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20201209151641139.png)
   
2. spring提供了两种容器类型:BeanFactory和ApplicationContext

   - BeanFactory:基础类型IOC容器,提供完整的Ioc服务支持.默认采用延迟初始化策略(懒加载).启动时间快,需要的系统资源少.
   - ApplicationContext:在BeanFactory上构建,;对其进行了功能上的升级,比如事件发布,国际化信息支持等.默认启动时完成所有初始化,需要更多的系统资源.
   - ![image-20201209152151032](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20201209152151032.png)

3. BeanFactory,BeanDefinitionRegister以及DefaultListtableBeanFactory的关系.

   1. BeanFactory 只是一个接口，我们最终需要一个该接口的实现来进行实际的Bean的管理， Default-
      ListableBeanFactory 就是这么一个比较通用的 BeanFactory 实现类。 DefaultListableBean-
      Factory 除了间接地实现了 BeanFactory 接口，还实现了 BeanDefinitionRegistry 接口，该接口才
      是在 BeanFactory 的实现中担当Bean注册管理的角色。基本上， BeanFactory 接口只定义如何访问容
      器内管理的Bean的方法，各个 BeanFactory 的具体实现类负责具体Bean的注册以及管理工作。
      BeanDefinitionRegistry 接口定义抽象了Bean的注册逻辑。通常情况下，具体的 BeanFactory 实现
      类会实现这个接口来管理Bean的注册。它们之间的关系如图4-3所示

   ![image-20201209160039061](C:\Users\b9082\AppData\Roaming\Typora\typora-user-images\image-20201209160039061.png)

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
   - 注解方式.注解注入其实是使用注解的方式进行构造器注入或者set注入。使用哪种方式注入,通过位置来判断,如果@autowired在构造方法上就是使用了构造器注入的.**这里需要区分Bean注入的方式和Bean注册方式的区别.**注解的方式其实是Bean注册方式的一种.
     - Bean的scope
       - 单例 与IOC容器拥有相同的寿命.
       - 多例  容器为每次请求创建一个对象 ,随后不在管理他的生命周期.
       - request,session,global session:request是多例的一个特例,只是拥有了具体的使用场景.
     - FactoryBean本质上一个Bean,只不过这种类型的bean本身,就是生产对象的工厂.不要与BeanFactory相混淆.

5. 小心prototype(多例的陷阱),有时候即使我们配置了多例,但是程序取对象的时候只取同一个对象,这是因为引用不当,每次请求没有向IOC的容器中获取对象,

   1. 需要配置lookup-method方法注入的方式.

   ```xml
   <bean id="persister" class="">
   	<lookup-method name  bean></lookup-method>
   </bean>
   ```

   2. 使用BeanFactoryAware接口
   3. 方法替换.

6. Spring的IOC容器

   1. 容器的启动阶段

      ​		通过BeanDefinitionReader对加载的Configuration进行解析和分析,并把信息编为BeanDefinition,然后将所有的BeanDefinition注册到相应的BeanDefinitionRegistry中.

   2. Bean的实例化阶段

      ​		当某个请求通过容器的getBean方法(或隐士的调用)明确的请求某个对象的时候,根据BeanDefinitionRegistry实例化Bean,注入依赖.

7. 容器启动时候的扩展

   - BeanFactoryPostProcess的容器扩展机制.允许容器实例化之前,对注册到容器BeanDefinition所保存的信息做相应的修改.例如修改Bean定义的某些属性,为Bean定义添加其他的信息等.
   - PropertyPlaceholderConfigurer

8. Bean的一生

   1. 对于BeanFactory使用的是Bean的懒加载策略,只到A被请求bean或者间接请求,间接是指有依赖到Bean时候.
   2. 虽然是通过BeanDefinition取得实例化信息,通过反射就能创建对象实例,但是并不是直接返回的对象实例,而是BeanWrapper对构造完成的对象实例进行包裹.返回相应的BeanWrapper.
   3. BeanWarpperImpl实现类作用是对每个Bean实现包裹,设置或者获取Bean的属性,BeanWarpperImpl间接继承了PropertyEditorRegitry,会将注册信息传递给wrapper.

9. BeanPostProcessor

   1. 存在于对象实例化阶段,注意与BeanFactorPostProcess则是存在于容器的启动阶段.
   2. 各种aware接口就是在这postProcessBeforeInitialization处理的.
   3. Spring的AOP功能就是用BeanPostProcess来为对象生成相应的代理对象.

10. 自定义BeanPostProcess

   1. 实现BeanPostProcessor接口,重写BeforeInitialization
   2. 将自定义的processor注册到容器.
      - 对于BeanFactory使用configurableBeanFactory.addBeanProcessor方法
      - 对于Application则作为一个Bean配置在xml中就可以了.

11. Bean的销毁

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

1. @Autowired  @Quality @ Resource 
2. @PostConstruct和@PreDestroy不是服务于依赖注入的,而是生命周期相关的方法,这与Spring的InitializingBean和DisposableBean接口,以及配置项中,init-method和destory-method起到类似的作用.
3. 使用注解其实是类似使用自定义的BeanPostProcess,需要将注解对应的AutowiredAnnotationBeanPostProcessor,必须在配置文件中使用<context:annotation-config>配置开启注解.,开启包扫描的注解<context:component-scan>也注入了这个process.

### 第七章 AOP

1. 动态的Aop和静态的Aop,动态Aop是学习的重点,动态Aop的织入过程是在系统运行开始之后进行的,而不是预先编译到系统类中,这种方式更容易开发和集成,缺点是需要性能的支持.
2. Aop的基本概念:
   - 切点(jointPoint):需要对方法增强的地方.可以在异常,方法构造任何地方进行增强的点.
   - pointcut:是jointPoint的表达式.与切点的区别是切点是在方法内的某个点,只是描述,与执行无关..pointcut才是定义了方法增强的位置.
   - advice:通知 相当于执行的方法体.
3. 静态代理没遇见一个类就需要生成一个代理,所以一般使用动态的代理,主要由一个类和一个接口组成,Proxy类和InvocationHandler接口.
4. 常见的PonitCut
   - NameMatchMethodPointCut
     - 属于StaticMethodMatcherPointcut的子类,可以根据自身指定的方法名和Jointpoint处的方法名称进行匹配.但是无法对重载的方法进行匹配.
5. 前置通知可以用来检测文件的位置是否存在.
6. 异常后置通知可以第一时间知道报出异常