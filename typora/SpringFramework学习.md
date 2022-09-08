## 第一章 IOC容器

1. Bean是程序的骨架,Spring IOC 管理着这些Bean,Bean的层级关系反映到容器使用的配置元数据中.

2. org.springframework.beans 和 org.springframework.context 是 Spring IOC 的基础.

3. 需要通过注解或者xml的形式定义Bean,定义的类与容器真正使用的类相对应.类可以相互依赖.

   1. ~~~xml
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

4. 定义后可以通过ClassPathXmlApplicationContext获取到

   1. ~~~java
      // create and configure beans
      ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
      
      // retrieve configured instance
      PetStoreService service = context.getBean("petStore", PetStoreService.class);
      
      // use configured instance
      List<String> userList = service.getUsernameList();
      ~~~

5. 在Spring文档中，“factory bean”指的是在Spring容器中配置的bean，它通过实例或静态工厂方法创建对象。相比之下，FactoryBean(注意大写)指的是spring特定的FactoryBean实现类。

6. 控制反转又名依赖注入,原本Bean的实例化需要Bean层层的构建依赖的Bean,现在由容器构建完成,可直接使用Bean."获得依赖对象的过程被反转了",控制被反转之后，获得依赖对象的过程由自身管理变为了由IOC容器主动注入.所谓依赖注入，就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。