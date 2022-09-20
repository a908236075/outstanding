# 1. IOC容器

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

5. 当依赖关系并不十分直接的时候可以用depend-on属性,多依赖.

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


   7. 方法注入:当依赖的类不是单例的,而想从容器中拿到单例类时候,可以实现ApplicationContextAware,调用getBean的方法.

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
