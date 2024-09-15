---

title: SpringBoot学习笔记
date: 2024-08-11 11:00:00 +0800
categories: [Springboot]
tags: [SpringBoot]

---
果然最好的记忆方法就是记笔记学习，之前`springboot`已经学过一次了，回头一看全忘记了。

本篇文章只会记录一些我看 `https://www.yuque.com/leifengyang/springboot3/vznmdeb4kgn90vrx`这个文档中个人认为比较重要的点。



### spring必备概念

#### 理解`SSM`

首先理解什么是 `ssm`: `SSM` = `spring` + `spring MVC` + `Mybatis`

**Spring**：一个全面的应用程序框架，提供 `IOC`（控制反转）和 `AOP`（面向切面编程）等功能，简化了 Java 应用的开发。Spring 也是 SSM 组合的核心，提供了基础架构的支持。

**Spring MVC**：是 Spring 框架的 Web 模块，实现了 `MVC` 设计模式，用于开发 Web 应用中的控制层和视图层。

+ **Model（模型）**：处理数据和业务逻辑的部分。它代表应用程序的数据结构，直接管理数据、逻辑和规则。
+ **View（视图）**：负责用户界面的显示部分。它接收来自 Model 的数据，并将其呈现给用户，在前后端分离的项目中通常返回 `json`数据
+ **Controller（控制器）**：处理用户输入并转换为对 Model 的操作，决定将哪个 View 显示给用户。Controller 是 View 和 Model 之间的桥梁。

**MyBatis**：一个持久层框架，用于简化数据库操作。它通过 SQL 映射文件或注解将 **Java 对象与数据库表进行映射**，负责数据的持久化。



### IOC容器（inversion                         of control 控制反转	）

我们要知道，对于常规的三层架构请求流程如下：

![image](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-1723592004802.png)

本质上，其是由一个个组件构成，因此我们需要实例化非常多的组件对象，而 `Spring`的一大特点，就是 `IOC`容器，这个容器就是替我们实例化、属性赋值、管理存活周期；

注：组件与对象的关系

+ 组件一定是对象
+ 对象不一定是组件

综上所属，`Spring`充当一个组件容器，创建、管理、存储组件，减少我们管理、创建的编码压力；而程序员为此需要做的就是提供配置原数据（以XML、Java注解、Java代码三种形式表现），允许表达组成应用程序组件之间丰富的依赖关系。

1. XML配置方式：是Spring框架最早的配置方式之一，通过在XML文件中定义Bean及其依赖关系、Bean的作用域等信息，让Spring IoC容器来管理Bean之间的依赖关系。该方式从Spring框架的第一版开始提供支持。
2. 注解方式：从Spring 2.5版本开始提供支持，可以通过在Bean类上使用注解来代替XML配置文件中的配置信息。通过在Bean类上加上相应的注解（如@Component, @Service, @Autowired等），将Bean注册到Spring IoC容器中，这样Spring IoC容器就可以管理这些Bean之间的依赖关系。
3. **Java配置类**方式：从Spring 3.0版本开始提供支持，通过Java类来定义Bean、Bean之间的依赖关系和配置信息，从而代替XML配置文件的方式。Java配置类是一种使用Java编写配置信息的方式，通过@Configuration、@Bean等注解来实现Bean和依赖关系的配置。

#### 代码实现

**SpringIoc容器接口**： 

`BeanFactory` 接口提供了一种高级配置机制，能够管理任何类型的对象，它是SpringIoC容器标准化超接口！

`ApplicationContext` 是 `BeanFactory` 的子接口。它扩展了以下功能：

- 更容易与 Spring 的 AOP 功能集成
- 消息资源处理（用于国际化）
- 特定于应用程序给予此接口实现，例如Web 应用程序的 `WebApplicationContext`

简而言之， `BeanFactory` 提供了配置框架和基本功能，而 `ApplicationContext` 添加了更多特定于企业的功能。 `ApplicationContext` 是 `BeanFactory` 的完整超集！

**ApplicationContext容器实现类**：

| 类型名                             | 简介                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| ClassPathXmlApplicationContext     | 通过读取类路径下的 XML 格式的配置文件创建 IOC 容器对象       |
| FileSystemXmlApplicationContext    | 通过文件系统路径读取 XML 格式的配置文件创建 IOC 容器对象     |
| AnnotationConfigApplicationContext | 通过读取Java配置类创建 IOC 容器对象                          |
| WebApplicationContext              | 专门为 Web 应用准备，基于 Web 环境创建 IOC 容器对象，并将对象引入存入 ServletContext 域中。 |

#### 基于xml配置

以xml配置为例，实例化`ioc`容器：

```java
//实例化ioc容器,读取外部配置文件,最终会在容器内进行ioc和di动作
ApplicationContext context = 
           new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

货期组件对象：

```java
//创建ioc容器对象，指定配置文件，ioc也开始实例组件对象
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
//获取ioc容器的组件对象
PetStoreService service = context.getBean("petStore", PetStoreService.class);
//使用组件对象
List<String> userList = service.getUsernameList();
```

##### （组件周期方法）

1. 周期方法概念

    我们可以在组件类中定义方法，然后当IoC容器实例化和销毁组件对象的时候进行调用！这两个方法我们成为生命周期方法！

    类似于Servlet的init/destroy方法,我们可以在周期方法完成初始化和释放资源等工作。
2. 周期方法声明

```Java
public class BeanOne {

  //周期方法要求： 方法命名随意，但是要求方法必须是 public void 无形参列表
  public void init() {
    // 初始化逻辑
  }
}

public class BeanTwo {

  public void cleanup() {
    // 释放资源逻辑
  }
}
```
3. 周期方法配置

```XML
<beans>
  <bean id="beanOne" class="examples.BeanOne" init-method="init" />
  <bean id="beanTwo" class="examples.BeanTwo" destroy-method="cleanup" />
</beans>
```
##### （组件作用域配置）

1. Bean作用域概念

    `<bean` 标签声明Bean，只是将Bean的信息配置给SpringIoC容器！

    在IoC容器中，这些`<bean`标签对应的信息转成Spring内部 `BeanDefinition` 对象，`BeanDefinition` 对象内，包含定义的信息（id,class,属性等等）！

    这意味着，`BeanDefinition`与`类`概念一样，SpringIoC容器可以可以根据`BeanDefinition`对象反射创建多个Bean对象实例。

    具体创建多少个Bean的实例对象，由Bean的作用域Scope属性指定！
2. 作用域可选值

| 取值      | 含义                                        | 创建对象的时机   | 默认值 |
| --------- | ------------------------------------------- | ---------------- | ------ |
| singleton | 在 IOC 容器中，这个 bean 的对象始终为单实例 | IOC 容器初始化时 | 是     |
| prototype | 这个 bean 在 IOC 容器中有多个实例           | 获取 bean 时     | 否     |

如果是在WebApplicationContext环境下还会有另外两个作用域（但不常用）：

| 取值    | 含义                 | 创建对象的时机 | 默认值 |
| ------- | -------------------- | -------------- | ------ |
| request | 请求范围内有效的实例 | 每次请求       | 否     |
| session | 会话范围内有效的实例 | 每次会话       | 否     |

3. 作用域配置

    配置scope范围

```XML
<!--bean的作用域 
    准备两个引用关系的组件类即可！！
-->
<!-- scope属性：取值singleton（默认值），bean在IOC容器中只有一个实例，IOC容器初始化时创建对象 -->
<!-- scope属性：取值prototype，bean在IOC容器中可以有多个实例，getBean()时创建对象 -->
<bean id="happyMachine8" scope="prototype" class="com.atguigu.ioc.HappyMachine">
    <property name="machineName" value="happyMachine"/>
</bean>

<bean id="happyComponent8" scope="singleton" class="com.atguigu.ioc.HappyComponent">
    <property name="componentName" value="happyComponent"/>
</bean>
```
##### DI（dependency injection）

DI 是指在组件之间传递依赖关系的过程中，将依赖关系在容器内部进行处理，这样就不必在应用程序代码中硬编码对象之间的依赖关系，实现了对象之间的解耦合。在 Spring 中，DI 是通过 XML **配置文件或注解** 的方式实现的。它提供了三种形式的依赖注入：构造函数注入、Setter 方法注入和接口注入 。

主流的依赖注入方法包含：`setter`方法注入和构造函数方法注入，注入分为引用类型和基本类型；

##### setter 注入

```xml
<bean id="simpleMovieLister" class="examples.SimpleMovieLister">
  <!-- setter方法，注入movieFinder对象的标识id
       name = 属性名  ref = 引用bean的id值
   -->
  <property name="movieFinder" ref="movieFinder" />

  <!-- setter方法，注入基本数据类型movieName
       name = 属性名 value= 基本类型值
   -->
  <property name="movieName" value="消失的她"/>
</bean>

<bean id="movieFinder" class="examples.MovieFinder"/>

```

##### 构造函数注入

```xml
<!-- 场景1: 多参数，可以按照相应构造函数的顺序注入数据 -->
<beans>
  <bean id="userService" class="x.y.UserService">
    <!-- value直接注入基本类型值 -->
    <constructor-arg  value="18"/>
    <constructor-arg  value="赵伟风"/>
    
    <constructor-arg  ref="userDao"/>
  </bean>
  <!-- 被引用类bean声明 -->
  <bean id="userDao" class="x.y.UserDao"/>
</beans>


<!-- 场景2: 多参数，可以按照相应构造函数的名称注入数据 -->
<beans>
  <bean id="userService" class="x.y.UserService">
    <!-- value直接注入基本类型值 -->
    <constructor-arg name="name" value="赵伟风"/>
    <constructor-arg name="userDao" ref="userDao"/>
    <constructor-arg name="age"  value="18"/>
  </bean>
  <!-- 被引用类bean声明 -->
  <bean id="userDao" class="x.y.UserDao"/>
</beans>

<!-- 场景2: 多参数，可以按照相应构造函数的角标注入数据 
           index从0开始 构造函数(0,1,2....)
-->
<beans>
    <bean id="userService" class="x.y.UserService">
    <!-- value直接注入基本类型值 -->
    <constructor-arg index="1" value="赵伟风"/>
    <constructor-arg index="2" ref="userDao"/>
    <constructor-arg index="0"  value="18"/>
  </bean>
  <!-- 被引用类bean声明 -->
  <bean id="userDao" class="x.y.UserDao"/>
</beans>

```

#### 基于注解配置

和 XML 配置文件一样，注解本身并不能执行，注解本身仅仅只是做一个标记，具体的功能是框架检测到注解标记的位置，然后针对这个位置按照注解标记的功能来执行具体操作。

本质上：所有一切的操作都是 Java 代码来完成的，XML 和注解只是告诉框架中的 Java 代码如何执行。

##### 组件标记注解：

+ `@Component`
+ `@Repository`
+ `@Service`
+ `@Controller`

##### 周期方法注解

+ `@posConstruct`（后置事件）
+ `@preDestroy`（前置事件）、

```java
public class BeanOne {

  //周期方法要求： 方法命名随意，但是要求方法必须是 public void 无形参列表
  @PostConstruct  //注解制指定初始化方法
  public void init() {
    // 初始化逻辑
  }
}

public class BeanTwo {
  
  @PreDestroy //注解指定销毁方法
  public void cleanup() {
    // 释放资源逻辑
  }
}
```

##### 组件作用域

`@scope(scopeName=ConfigurableBeanFactory.SCOPE_SINGLETON)`

```java
@Scope(scopeName = ConfigurableBeanFactory.SCOPE_SINGLETON) //单例,默认值
@Scope(scopeName = ConfigurableBeanFactory.SCOPE_PROTOTYPE) //多例  二选一
public class BeanOne {

  //周期方法要求： 方法命名随意，但是要求方法必须是 public void 无形参列表
  @PostConstruct  //注解制指定初始化方法
  public void init() {
    // 初始化逻辑
  }
}
```

##### 自动装配（DI）

###### `@Autowired`

+ 参与自动装配的组件必须在`IOC`容器中
+ 成员变量直接标记 `@AutoWired`注解即可，不需要提供 `get`、`set`方法

**原理：**

- 首先根据所需要的组件类型到 IOC 容器中查找
    - 能够找到唯一的 bean：直接执行装配
    - 如果完全找不到匹配这个类型的 bean：装配失败
    - 和所需类型匹配的 bean 不止一个
        - 没有 `@Qualifier` 注解：根据 `@Autowired` 标记位置成员变量的变量名作为 bean 的 id 进行匹配
            - 能够找到：执行装配
            - 找不到：装配失败
        - 使用 `@Qualifier` 注解：根据 `@Qualifier` 注解中指定的名称作为 bean 的id进行匹配
            - 能够找到：执行装配
            - 找不到：装配失败

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/img018.png) 

举例：

```java
@Controller(value = "tianDog")
public class SoldierController {
    
    @Autowired
    @Qualifier(value = "maomiService222")
    // 根据面向接口编程思想，使用接口类型引入Service组件
    private ISoldierService soldierService;
```



```java
@Controller(value = "tianDog")
public class SoldierController {
    
    @Autowired
    private SoldierService soldierService;
    
    public void getMessage() {
        soldierService.getMessage();
    }
    
}
```

当然，这个注解不一定只给成员变量使用，也可以给**构造函数**或 `getter`函数使用：

```java
@Controller(value = "tianDog")
public class SoldierController {
    
    private SoldierService soldierService;
    
    @Autowired
    public SoldierController(SoldierService soldierService) {
        this.soldierService = soldierService;
    }
 
    //===============================
@Controller(value = "tianDog")
public class SoldierController {

    private SoldierService soldierService;

    @Autowired
    public void setSoldierService(SoldierService soldierService) {
        this.soldierService = soldierService;
    }

```

###### @Resource

- JSR-250 @Resource注解

    @Resource注解也可以完成属性注入。那它和@Autowired注解有什么区别？

    - @Resource注解是JDK扩展包中的，也就是说属于JDK的一部分。所以该注解是标准注解，更加具有通用性。(JSR-250标准中制定的注解类型。JSR是Java规范提案。)
    - @Autowired注解是Spring框架自己的。
    - **@Resource注解默认根据Bean名称装配，未指定name时，使用属性名作为name。通过name找不到的话会自动启动通过类型装配。**
    - **@Autowired注解默认根据类型装配，如果想根据名称装配，需要配合@Qualifier注解一起用。**
    - @Resource注解用在属性上、setter方法上。
    - @Autowired注解用在属性上、setter方法上、构造方法上、构造方法参数上。

    @Resource注解属于JDK扩展包，所以不在JDK当中，需要额外引入以下依赖：【**高于JDK11或低于JDK8需要引入以下依赖**】

```XML
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
```
- 使用

```Java
@Controller
public class XxxController {
    /**
     * 1. 如果没有指定name,先根据属性名查找IoC中组件xxxService
     * 2. 如果没有指定name,并且属性名没有对应的组件,会根据属性类型查找
     * 3. 可以指定name名称查找!  @Resource(name='test') == @Autowired + @Qualifier(value='test')
     */
    @Resource
    private XxxService xxxService;

    //@Resource(name = "指定beanName")
    //private XxxService xxxService;

    public void show(){
        System.out.println("XxxController.show");
        xxxService.show();
    }
}
```
##### Bean属性赋值

`@Value` 通常用于注入外部化属性

**声明外部配置**

application.properties

```Java
catalog.name=MovieCatalog
```

**xml引入外部配置**

```Java
<!-- 引入外部配置文件-->
<context:property-placeholder location="application.properties" />
```

**@Value注解读取配置**

```Java
package com.atguigu.components;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 * projectName: com.atguigu.components
 *
 * description: 普通的组件
 */
@Component
public class CommonComponent {

    /**
     * 情况1: ${key} 取外部配置key对应的值!
     * 情况2: ${key:defaultValue} 没有key,可以给与默认值
     */
    @Value("${catalog:hahaha}")
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```
#### 基于 配置类 配置

其实一句话总结就是：完全使用 注解方式进行配置，不再使用 `xml`文件方式了。

首先我们来回顾一下传统的 `xml`文件  +  注解方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


    <!-- 配置自动扫描的包 -->
    <!-- 1.包要精准,提高性能!
         2.会扫描指定的包和子包内容
         3.多个包可以使用,分割 例如: com.atguigu.controller,com.atguigu.service等
    -->
    <context:component-scan base-package="com.atguigu.components"/>

    <!-- 引入外部配置文件-->
    <context:property-placeholder location="application.properties" />
</beans>
```

测试创建`IOC`容器：

```java
 // xml方式配置文件使用ClassPathXmlApplicationContext容器读取
 ApplicationContext applicationContext =
                new ClassPathXmlApplicationContext("application.xml");
```

配置类 + 注解方式：

配置类：使用 `@Configuration`将一个普通类标记为`Spring`配置类

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

//标注当前类是配置类，替代application.xml    
@Configuration
//使用注解读取外部配置，替代 <context:property-placeholder标签
@PropertySource("classpath:application.properties")
//使用@ComponentScan注解,可以配置扫描包,替代<context:component-scan标签
@ComponentScan(basePackages = {"com.atguigu.components"})
public class MyConfiguration {
    
}
```

测试创建`IOC`容器：

```java
// AnnotationConfigApplicationContext 根据配置类创建 IOC 容器对象
ApplicationContext iocContainerAnnotation = 
new AnnotationConfigApplicationContext(MyConfiguration.class);
```

小总结：

+ @Configuration指定一个类为配置类，可以添加配置注解，替代配置xml文件
+ @ComponentScan(basePackages = {"包","包"}) 替代<context:component-scan标签实现注解扫描
+ @PropertySource("classpath:配置文件地址") 替代 <context:property-placeholder标签

实战：将一个第三方 Jar 包中的类实例化 装入 `IOC`容器

当然我们可以使用`xml`的方式如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">


    <!-- 引入外部属性文件 -->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!-- 实验六 [重要]给bean的属性赋值：引入外部属性文件 -->
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

</beans>
```

但是，官方更加推荐我们使用 配置类的方式来实现：

```java
//标注当前类是配置类，替代application.xml    
@Configuration
//引入jdbc.properties文件
@PropertySource({"classpath:application.properties","classpath:jdbc.properties"})
@ComponentScan(basePackages = {"com.atguigu.components"})
public class MyConfiguration {

    //如果第三方类进行IoC管理,无法直接使用@Component相关注解
    //解决方案: xml方式可以使用<bean标签
    //解决方案: 配置类方式,可以使用方法返回值+@Bean注解
    @Bean
    public DataSource createDataSource(@Value("${jdbc.user}") String username,
                                       @Value("${jdbc.password}")String password,
                                       @Value("${jdbc.url}")String url,
                                       @Value("${jdbc.driver}")String driverClassName){
        //使用Java代码实例化
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driverClassName);
        //返回结果即可
        return dataSource;
    }
}
```

###### @Bean

`@Bean` 注释注释方法。使用此方法在指定为方法返回值的类型的 `ApplicationContext` 中注册 Bean 定义。缺省情况下，Bean 名称与方法名称相同。下面的示例演示 `@Bean` 方法声明：

```Java
@Configuration
public class AppConfig {

  @Bean
  public TransferServiceImpl transferService() {
    return new TransferServiceImpl();
  }
}
```

前面的配置完全等同于下面的Spring XML：

```Java
<beans>
  <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```
###### 指定生命周期函数：

```java
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
```

###### 作用域scope

```java
@Configuration
public class MyConfiguration {

  @Bean
  @Scope("prototype")
  public Encryptor encryptor() {
    // ...
  }
}
```

##### @Import

  `@Import` 注释允许从另一个配置类加载 `@Bean` 定义，如以下示例所示：

```Java
@Configuration
public class ConfigA {

  @Bean
  public A a() {
    return new A();
  }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

  @Bean
  public B b() {
    return new B();
  }
}
```

  现在，在实例化上下文时不需要同时指定 `ConfigA.class` 和 `ConfigB.class` ，只需显式提供 `ConfigB` ，如以下示例所示：

```Java
public static void main(String[] args) {
  ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

  // now both beans A and B will be available...
  A a = ctx.getBean(A.class);
  B b = ctx.getBean(B.class);
}
```

  此方法简化了容器实例化，因为只需要处理一个类，而不是要求您在构造期间记住可能大量的 `@Configuration` 类。

##### 实战

```java
@Configuration
@ComponentScan(basePackages = "com.atguigu")
@PropertySource("classpath:jdbc.properties")
public class JavaConfig {

    @Value("${atguigu.url}")
    private String url;
    @Value("${atguigu.driver}")
    private String driver;
    @Value("${atguigu.username}")
    private String username;
    @Value("${atguigu.password}")
    private String password;

    @Bean(destroyMethod = "close")
    public DruidDataSource dataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driver);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }

    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource){
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

}
```

#### 4.5 三种配置方式总结

  ##### 4.5.1 XML方式配置总结
    1. 所有内容写到xml格式配置文件中
    2. 声明bean通过<bean标签
         3. < bean标签包含基本信息（id,class）和属性信息 <property name value / ref
    3. 引入外部的properties文件可以通过<context:property-placeholder
    4. IoC具体容器实现选择ClassPathXmlApplicationContext对象

  ##### 4.5.2 XML+注解方式配置总结
  1. 注解负责标记IoC的类和进行属性装配
  2. xml文件依然需要，需要通过<context:component-scan标签指定注解范围
  3. 标记IoC注解：@Component,@Service,@Controller,@Repository 
  4. 标记DI注解：@Autowired @Qualifier @Resource @Value
  5. IoC具体容器实现选择ClassPathXmlApplicationContext对象

  ##### 4.5.3 完全注解方式配置总结
  1. 完全注解方式指的是去掉xml文件，使用配置类 + 注解实现
  2. xml文件替换成使用@Configuration注解标记的类
  3. 标记IoC注解：@Component,@Service,@Controller,@Repository 
  4. 标记DI注解：@Autowired @Qualifier @Resource @Value
  5. <context:component-scan标签指定注解范围使用@ComponentScan(basePackages = {"com.atguigu.components"})替代
  6. <context:property-placeholder引入外部配置文件使用@PropertySource({"classpath:application.properties","classpath:jdbc.properties"})替代
  7. <bean 标签使用@Bean注解和方法实现
  8. IoC具体容器实现选择AnnotationConfigApplicationContext对象

### AOP（面向切面编程）

需求：不改变每个业务逻辑具体代码的前提下，补充业务逻辑流程；

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/aopimg002.webp) 

#### 代理模式

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/aopimg004.webp)![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/aopimg005.webp)

#####   （1）静态代理

```java
public class CalculatorStaticProxy implements Calculator {
    
    // 将被代理的目标对象声明为成员变量
    private Calculator target;
    
    public CalculatorStaticProxy(Calculator target) {
        this.target = target;
    }
    
    @Override
    public int add(int i, int j) {
    
        // 附加功能由代理类中的代理方法来实现
        System.out.println("参数是：" + i + "," + j);
    
        // 通过目标对象来实现核心业务逻辑
        int addResult = target.add(i, j);
    
        System.out.println("方法内部 result = " + result);
    
        return addResult;
    }
    ……
```

由于代码都写死了，完全不具备任何的灵活性。就拿日志功能来说，将来其他地方也需要附加日志，那还得再声明更多个静态代理类，那就产生了大量重复的代码，日志功能还是分散的，没有统一管理。 

##### （2）动态代理

动态代理技术分类

- JDK动态代理：JDK原生的实现方式，需要被代理的目标类必须**实现接口**！他会根据目标类的接口动态生成一个代理对象！代理对象和目标对象有相同的接口！（**拜把子**），proxy与被代理类实现同一个接口
- cglib：通过继承被代理的目标类实现代理，所以不需要目标类实现接口！（**认干爹**）proxy继承代理类

这两种动态代理技术好是好，就是程序员所需要写的代码还是太多了，不方便。

#### AOP介绍

AOP技术，它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。 可以在不修改原来代码的基础上添加新功能 

#### AOP应用

事务处理：在数据库操作中使用事务可以保证数据的一致性，可以使用AOP来实现事务处理的功能，可以在方法开始前开启事务，在方法执行完毕后提交或回滚事务。 

1. **日志记录**：在系统中记录日志是非常重要的，可以使用AOP来实现日志记录的功能，可以在方法执行前、执行后或异常抛出时记录日志。
2. **事务处理**：在数据库操作中使用事务可以保证数据的一致性，可以使用AOP来实现事务处理的功能，可以在方法开始前开启事务，在方法执行完毕后提交或回滚事务。
3. **安全控制**：在系统中包含某些需要安全控制的操作，如登录、修改密码、授权等，可以使用AOP来实现安全控制的功能。可以在方法执行前进行权限判断，如果用户没有权限，则抛出异常或转向到错误页面，以防止未经授权的访问。
4. **性能监控**：在系统运行过程中，有时需要对某些方法的性能进行监控，以找到系统的瓶颈并进行优化。可以使用AOP来实现性能监控的功能，可以在方法执行前记录时间戳，在方法执行完毕后计算方法执行时间并输出到日志中。
5. **异常处理**：系统中可能出现各种异常情况，如空指针异常、数据库连接异常等，可以使用AOP来实现异常处理的功能，在方法执行过程中，如果出现异常，则进行异常处理（如记录日志、发送邮件等）。
6. **缓存控制**：在系统中有些数据可以缓存起来以提高访问速度，可以使用AOP来实现缓存控制的功能，可以在方法执行前查询缓存中是否有数据，如果有则返回，否则执行方法并将方法返回值存入缓存中。
7. **动态代理**：AOP的实现方式之一是通过动态代理，可以代理某个类的所有方法，用于实现各种功能。

### 声明式事务

#### 传统编程式事务：

```java
Connection conn = ...;
  
try {
    // 开启事务：关闭事务的自动提交
    conn.setAutoCommit(false);
    // 核心操作
    // 业务代码
    // 提交事务
    conn.commit();
  
}catch(Exception e){
  
    // 回滚事务
    conn.rollBack();
  
}finally{
  
    // 释放数据库连接
    conn.close();
  
}
```

编程式的实现方式存在缺陷：

- 细节没有被屏蔽：具体操作过程中，所有细节都需要程序员自己来完成，比较繁琐。
- 代码复用性不高：如果没有有效抽取出来，每次实现功能都需要自己编写代码，代码就没有得到复用。

#### 声明式事务

我们首先了解一下什么是事务：

**事务（Transaction）**：事务是一个操作的序列，这些操作要么全部成功，要么全部失败。如果事务中的某个操作失败，则已经执行的所有操作必须回滚，以确保数据库的完整性和一致性。

Spring 的声明式事务是通过 AOP 机制来实现的。`@Transactional` 注解会被 Spring 的事务拦截器（TransactionInterceptor）检测到，拦截器会在方法调用前开启事务，在方法调用后根据情况提交或回滚事务。

`@Transactional` 注解在 Spring Boot 中通常用于服务层（Service Layer）。这是因为服务层负责业务逻辑的处理，其中涉及到多个数据访问操作的组合，这些操作通常需要在一个事务中执行，以确保数据的一致性和完整性。

##### 依赖

- spring-tx: 包含声明式事务实现的基本规范（事务管理器规范接口和事务增强等等）
- spring-jdbc: 包含DataSource方式事务管理器实现类DataSourceTransactionManager
- spring-orm: 包含其他持久层框架的事务管理器实现类例如：Hibernate/Jpa等

##### 事务管理器

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-1723969152035.png) 

`DataSourceTransactionManager`类中的主要方法：

- `doBegin`()：开启事务
- `doSuspend`()：挂起事务
- `doResume`()：恢复挂起的事务
- `doCommit`()：提交事务
- `doRollback`()：回滚事务

`@Transactional`注解既可以使用在类上，也可以使用在方法上。

如果你的应用使用的是 JDBC 或 MyBatis，Spring Boot 会自动配置并注入一个 `DataSourceTransactionManager`（自动转配机制）

事务传播行为(Propagation)：

传播行为定义了一个事务方法在调用另一个事务方法时，事务如何传播。常见的传播行为包括：

- **REQUIRED**：使用当前事务，如果没有事务则创建一个新事务。
- **REQUIRES_NEW**：总是创建一个新事务，挂起当前事务。
- **NESTED**：如果当前事务存在，则在嵌套事务中执行。

代码举例什么是事务传递：

```java
@Transactional
public void MethodA(){
    // ...
    MethodB();
    // ...
}

//在被调用的子方法中设置传播行为，代表如何处理调用的事务！ 是加入，还是新事务等！
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void MethodB(){
    // ...
}

```

我们还需要详细了解一下创建新事物与加入事务的区别

**创建新的事务 (`REQUIRES_NEW`)**

- **定义**：每次该方法被调用时，无论是否已经存在一个事务，都会创建一个新的独立事务。如果当前存在事务，Spring 会暂时挂起这个现有事务，直到新的事务完成。
- **使用场景**：适用于某些操作必须独立于外部事务的场景。无论外部事务是否回滚，新的事务都不会受影响。

**示例**：

```java
@Service
public class PaymentService {

    @Autowired
    private PaymentRepository paymentRepository;

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void processRefund(Long orderId) {
        // 处理退款逻辑
        paymentRepository.refund(orderId);
        // 新事务，不受外部事务影响
    }
}

@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private PaymentService paymentService;

    @Transactional(propagation = Propagation.REQUIRED)
    public void cancelOrder(Long orderId) {
        // 取消订单逻辑
        orderRepository.cancel(orderId);

        // 处理退款
        paymentService.processRefund(orderId);
        // 无论取消订单事务是否回滚，退款事务都会提交
    }
}
```

**分析**：

- 当 `cancelOrder` 被调用时，Spring 会创建一个事务用于取消订单。如果在取消订单后，调用 `processRefund` 方法时，`processRefund` 会创建一个新的事务。即使 `cancelOrder` 的事务因某种原因回滚了，`processRefund` 中的事务依然会提交。因此，订单取消和退款操作是相互独立的。

**加入现有事务 (`REQUIRED`)**

- **定义**：如果当前已经存在一个事务，方法将加入到这个事务中；如果没有事务，则创建一个新的事务。这种传播方式是 Spring 的默认行为。
- **使用场景**：适用于多个操作需要在同一个事务中执行的场景，以确保操作的原子性。

**示例**：

```
java复制代码@Service
public class InventoryService {

    @Autowired
    private InventoryRepository inventoryRepository;

    @Transactional(propagation = Propagation.REQUIRED)
    public void decreaseStock(Long productId) {
        // 扣减库存逻辑
        inventoryRepository.decrease(productId);
    }
}

@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private InventoryService inventoryService;

    @Transactional(propagation = Propagation.REQUIRED)
    public void placeOrder(Long productId) {
        // 创建订单
        orderRepository.create(productId);

        // 扣减库存
        inventoryService.decreaseStock(productId);
        // 如果任一操作失败，整个事务回滚
    }
}
```

**分析**：

- 当 `placeOrder` 方法被调用时，Spring 会创建一个事务。`placeOrder` 中的所有操作，包括订单创建和库存扣减，都会在同一个事务中执行。如果其中任何一个操作失败，整个事务将回滚，确保数据一致性。

常用属性：

`@Transactional` 注解有多个可配置的属性，常用的有：

- **propagation**：事务传播行为，默认值为 `Propagation.REQUIRED`。常见的选项包括：
  - **REQUIRED**：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新事务。
  - **REQUIRES_NEW**：总是创建一个新事务，如果当前存在事务，则挂起当前事务。
  - **SUPPORTS**：如果当前存在事务，则加入该事务；如果没有事务，则以非事务方式继续执行。
- **isolation**：事务隔离级别，默认值为 `Isolation.DEFAULT`。常见的选项包括：
  - **DEFAULT**：使用底层数据库的默认隔离级别。
  - **READ_UNCOMMITTED**：最低的隔离级别，允许读取未提交的数据（脏读）。
  - **READ_COMMITTED**：确保读取的数据是已提交的。
  - **REPEATABLE_READ**：确保在同一个事务中多次读取数据时，读取到的数据是一致的。
  - **SERIALIZABLE**：最高的隔离级别，完全锁定相关数据，防止幻读。
- **timeout**：事务的超时时间，单位为秒。默认值为 -1，即没有超时限制。
- **readOnly**：标记事务是否为只读，默认值为 `false`。如果设置为 `true`，可以帮助数据库引擎优化事务。
- **rollbackFor** 和 **rollbackForClassName**：指定哪些异常会触发事务回滚。
- **noRollbackFor** 和 **noRollbackForClassName**：指定哪些异常不会触发事务回滚。

**例子**：

```java
@Transactional(propagation = Propagation.REQUIRES_NEW, isolation = Isolation.SERIALIZABLE, timeout = 30, readOnly = false)
public void updateUserAccount(String userId, double amount) {
    // 复杂的业务逻辑操作
}
```



### `maven`

https://www.wolai.com/fbnhGx8eE9JfZugFpbCWmC 这个文档写的非常详细，设计的点都有介绍

### 自动配置机制

#### 发现

```java
 public static void main(String[] args) {

        //java10： 局部变量类型的自动推断
        var ioc = SpringApplication.run(MainApplication.class, args);

        //1、获取容器中所有组件的名字
        String[] names = ioc.getBeanDefinitionNames();
        //2、挨个遍历：
        // dispatcherServlet、beanNameViewResolver、characterEncodingFilter、multipartResolver
        // SpringBoot把以前配置的核心组件现在都给我们自动配置好了。
        for (String name : names) {
            System.out.println(name);
        }

    }
```

通过以上代码我们可以轻易发现，`springboot`已经为我们在容器中自动配置好了各种组件，那么这些组件是怎么引入的呢，需要进一步分析



#### 整体流程

​	整体流程其实非常简单：

- **1、**导入`starter-web`：导入了web开发场景  （`spring-boot-autoconfigure` 包的导入）

- - 1、场景启动器导入了相关场景的所有依赖：`starter-json`、`starter-tomcat`、`springmvc`
  - 2、每个场景启动器都引入了一个`spring-boot-starter`，核心场景启动器。
  - 3、**核心场景启动器**引入了`spring-boot-autoconfigure`包。
  - 4、`spring-boot-autoconfigure`包里面囊括了所有场景的所有配置。
  - 5、只要这个包下的所有类都能生效，那么相当于SpringBoot官方写好的整合功能就生效了。
  - 6、SpringBoot默认却扫描不到 `spring-boot-autoconfigure`下写好的所有**配置类**。（这些**配置类**给我们做了整合操作），**默认只扫描主程序所在的包**。

- **2、****主程序**：`@SpringBootApplication` （自动配置包內的配置类 如何 生效）

- - 1、`@SpringBootApplication`由三个注解组成`@SpringBootConfiguration`、`@EnableAutoConfiguratio`、`@ComponentScan`

  - 2、SpringBoot默认只能扫描自己主程序所在的包及其下面的子包，扫描不到 `spring-boot-autoconfigure`包中官方写好的**配置类**

  - 3、`@EnableAutoConfiguration`：SpringBoot **开启自动配置的核心，**它告诉 Spring Boot 在应用启动时自动扫描并加载符合条件的 `@Configuration` 配置类，自动为应用程序配置所需的 `Bean`  

       下面对这个注解详细解释：

- - - `@Import(AutoConfigurationImportSelector.class)`提供功能：批量给容器中导入组件。`AutoConfigurationImportSelector`类会默认加载`spring-boot-autoconfigure`包下的 `META-INF/spring/**org.springframework.boot.autoconfigure.AutoConfiguration**.imports`指定的配置类（通过 `@Import` 注解，可以将这些本来不会被扫描到的类强制导入到 Spring 上下文中，从而使它们变成可管理的 Spring Bean。 

      ![1723278751877](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1723278751877.png)

    - 虽然导入了`142`个自动配置类

  - ![1723278780666](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1723278780666.png)

- - 4、按需生效：

- - - 并不是这`142`个自动配置类都能生效
    - 每一个自动配置类，都有条件注解`@ConditionalOnxxx`，只有条件成立，才能生效 

  - ![1723278834914](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1723278834914.png)

- **3、**`**xxxxAutoConfiguration**`**自动配置类**

- - **1、给容器中使用@Bean 放一堆组件。**

  - 2、每个**自动配置类**都可能有这个注解`@EnableConfigurationProperties(**ServerProperties**.class)`，用来把配置文件中配的指定前缀的属性值封装到 `xxxProperties`**属性类**中，每一个属性类 都与 一个前缀绑定：

    ![1723278940835](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1723278940835.png)

    另外配置类与属性类的文件关系如图：

    ![1723279172404](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1723279172404.png)

    

  - 3、以Tomcat为例：把服务器的所有配置都是以`server`开头的。配置都封装到了属性类中。

  - 4、给**容器**中放的所有**组件**的一些**核心参数**，都来自于`**xxxProperties**`**。**`**xxxProperties**`**都是和配置文件绑定。**

  - **只需要改配置文件的值，核心组件的底层参数都能修改**

  - 自动配置类都是用了 `AutoConfigration`注解，`@AutoConfiguration` 注解标识的类可以被 `@EnableAutoConfiguration` 机制扫描到，并根据条件注入到 Spring 上下文中。 

### 常用注解

#### 组件注解

用于定义配置、管理 Bean 的生命周期以及控制类的扫描和导入。 

##### 1. **@Configuration**

- **作用**：`@Configuration` 用于定义配置类。配置类相当于传统 XML 配置文件，主要用于定义 Bean 以及其他 Spring 应用的配置。

- **特点**：通常包含一个或多个 `@Bean` 方法，Spring 会将这些方法返回的对象注册为 Spring 上下文中的 Bean。

- **示例**：

  ```java
  
  @Configuration
  public class MyAppConfig {
  
      @Bean
      public MyService myService() {
          return new MyServiceImpl();
      }
  }
  ```

##### 2. **@SpringBootConfiguration**

- **作用**：`@SpringBootConfiguration` 是 Spring Boot 特有的注解，主要用于标记一个类为 Spring Boot 的配置类。

- **特点**：它是 `@Configuration` 的一种特化形式，实际上包含了 `@Configuration`，因此具备相同的功能。

- **典型用法**：通常与 `@SpringBootApplication` 一起使用，而 `@SpringBootApplication` 本身就包含了 `@SpringBootConfiguration`。

- **示例**：

  ```java
  
  @SpringBootConfiguration
  public class MySpringBootAppConfig {
      // 配置类代码
  }
  ```

##### 3. **@Bean**

- **作用**：`@Bean` 用于标注方法，该方法会返回一个对象，这个对象将被 Spring 容器管理，作为 Spring 上下文中的一个 Bean。

- **特点**：通常与 `@Configuration` 一起使用，用于显式地定义 Bean。

- **示例**：

  ```java
  @Configuration
  public class AppConfig {
  
      @Bean
      public DataSource dataSource() {
          return new HikariDataSource();
      }
  }
  ```

##### 4. **@Scope**

- **作用**：`@Scope` 用于定义 Spring Bean 的作用域，即控制 Spring 如何创建和管理这个 Bean 实例。

- **常见作用域**：

  - `singleton`（默认）：整个应用上下文中只有一个实例。
  - `prototype`：每次请求都会创建一个新的实例。
  - `request`：在 Web 应用中，每个 HTTP 请求对应一个实例。
  - `session`：在 Web 应用中，每个 HTTP Session 对应一个实例。

- **示例**：

  ```java
  @Bean
  @Scope("prototype")
  public MyBean myBean() {
      return new MyBean();
  }
  ```

##### 5. **@Controller**

- **作用**：`@Controller` 是 Spring MVC 中的注解，标记一个类为控制器，主要用于定义处理 HTTP 请求的端点（Web 层）。

- **特点**：通常与 `@RequestMapping` 一起使用，定义请求的映射路径。

- **示例**：

  ```java
  @Controller
  public class MyController {
  
      @RequestMapping("/hello")
      public String sayHello() {
          return "hello";
      }
  }
  ```

##### 6. **@Service**

- **作用**：`@Service` 用于标记业务逻辑层的组件，表示该类是一个服务类。Spring 会自动将其注册为 Spring 上下文中的 Bean。

- **特点**：这是一个语义化的注解，用于表示该类承担服务的角色。

- **示例**：

  ```java
  @Service
  public class MyService {
      public void execute() {
          // 业务逻辑
      }
  }
  ```

##### 7. **@Repository**

- **作用**：`@Repository` 用于标记持久层（DAO）组件，通常用于数据库操作相关的类。Spring 会自动将其注册为 Spring 上下文中的 Bean。

- **特点**：Spring 在 `@Repository` 注解的类上提供了持久化异常转换功能，将数据库相关的异常转换为 Spring 的数据访问异常。

- **示例**：

  ```java
  @Repository
  public class MyRepository {
      public List<MyEntity> findAll() {
          // 数据库查询
      }
  }
  ```

##### 8. **@Component**

- **作用**：`@Component` 是一个通用的注解，用于标记任何受 Spring 容器管理的组件。Spring 会自动将其注册为 Spring 上下文中的 Bean。

  - - 当类不符合更具体角色（如@Controller、@Service、@Repository）时，可以使用@Component来标识它们为Spring组件，使它们成为Spring容器管理的Bean。

- **特点**：它是 `@Service`、`@Repository` 和 `@Controller` 的基础注解。任何没有特定语义的类都可以使用 `@Component` 注解。当然，对比传统的`xml`注册方法，使用注解也可以设定 组件对象`id`，通过`value`属性就可以，例如： `@Component(value="myComponent")`

- **示例**：

  ```java
  @Component
  public class MyComponent {
      // 通用组件
  }
  ```

##### 9. **@Import**

- **作用**：`@Import` 用于将一个或多个配置类导入到当前配置类中，使得这些配置类的 Bean 定义可以被加载到 Spring 上下文中。

- **特点**：可以导入具体的类、选择器（`ImportSelector`）或 Bean 定义注册器（`ImportBeanDefinitionRegistrar`）。

- **示例**：

  ```java
  @Configuration
  @Import({AnotherConfig.class, YetAnotherConfig.class})
  public class MyConfig {
      // 当前配置类
  }
  ```

##### 10. **@ComponentScan**

- **作用**：`@ComponentScan` 用于指定 Spring 框架应该扫描哪些包以查找和注册带有 `@Component`、`@Service`、`@Repository`、`@Controller` 等注解的类作为 Spring Bean。

- **特点**：通常与 `@Configuration` 一起使用，可以指定基础包，或者使用过滤器来包括或排除某些类。

- **示例**：

  ```java
  @Configuration
  @ComponentScan(basePackages = "com.example.myapp")
  public class AppConfig {
      // 配置类，扫描 com.example.myapp 包
  }
  ```

#### 条件注解

如果注解指定的**条件成立**，则触发指定行为

**\*@ConditionalOnXxx***

**@ConditionalOnClass：如果类路径中存在这个类，则触发指定行为**

**@ConditionalOnMissingClass：如果类路径中不存在这个类，则触发指定行为**

**@ConditionalOnBean：如果容器中存在这个Bean（组件），则触发指定行为**

**@ConditionalOnMissingBean：如果容器中不存在这个Bean（组件），则触发指定行为**

场景：

- 如果存在`FastsqlException`这个类，给容器中放一个`Cat`组件，名cat01，
- 否则，就给容器中放一个`Dog`组件，名dog01
- 如果系统中有`dog01`这个组件，就给容器中放一个 User组件，名zhangsan 
- 否则，就放一个User，名叫lisi

**@ConditionalOnBean（value=组件类型，name=组件名字）：判断容器中是否有这个类型的组件，并且名字是指定的值**

以此类推，类似的条件注解还有很多



#### 属性绑定

`@ConfigurationProperties`： 声明组件的属性和配置文件哪些前缀开始项进行绑定，在属性类中使用

```java
@ConfigurationProperties(
    prefix = "server",
    ignoreUnknownFields = true
)
```



`@EnableConfigurationProperties`：快速注册注解：

- **场景：**用于将 `@ConfigurationProperties` 注解标记的类绑定到 Spring 环境（`Environment`）中的配置属性，并将这些类注册为 Spring 容器中的 Bean。 

```java
@EnableConfigurationProperties({ServerProperties.class}) // 绑定Properties类
@Import({BeanPostProcessorsRegistrar.class, ServletWebServerFactoryConfiguration.EmbeddedTomcat.class, ServletWebServerFactoryConfiguration.EmbeddedJetty.class, ServletWebServerFactoryConfiguration.EmbeddedUndertow.class})
public class ServletWebServerFactoryAutoConfiguration {
    public ServletWebServerFactoryAutoConfiguration() {
    }
}
```



将容器中任意**组件（Bean）的属性值**和**配置文件**的配置项的值**进行绑定**

- **1、给容器中注册组件（@Component、@Bean）**
- **2、使用****@ConfigurationProperties 声明组件和配置文件的哪些配置项进行绑定**

### YAML

YAML即“yet another makeup file”(“另一种标记语言”)

以 `.yml`或者 `.yaml`作为后缀

#### 基本语法

- **大小写敏感**
- 使用**缩进表示层级关系，k: v，使用空格分割k,v**
- 缩进时不允许使用Tab键，只允许**使用空格**。换行
- 缩进的空格数目不重要，只要**相同层级**的元素**左侧对齐**即可
- **# 表示注释**，从这个字符一直到行尾，都会被解析器忽略。
- `-` 符号表示一个列表（数组）的项 

支持的写法：

- **对象**：**键值对**的集合，如：映射（map）/ 哈希（hash） / 字典（dictionary）
- **数组**：一组按次序排列的值，如：序列（sequence） / 列表（list）
- **纯量**：单个的、不可再分的值，如：字符串、数字、bool、日期

关于引号的使用：

**无需引号的情况**：大多数情况下，普通字符串、数字、布尔值等都可以不使用引号。

**需要引号的情况**：当字符串包含特殊字符、空白、特殊值（如 `true`、`false`、`null`）、数字需要作为字符串处理等情况时，建议使用引号。

#### 举例

```java
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Date birthDay;
    private Boolean like;
    private Child child; 
    private List<Dog> dogs; 
    private Map<String, Cat> cats; 
}


@Data
public class Dog {
    private String name;
    private Integer age;
}

@Data
public class Child {
    private String name;
    private Integer age;
    private Date birthDay;
    private List<String> text; //数组
}

@Data
public class Cat {
    private String name;
    private Integer age;
}
```

`yaml`属性表示：

```yaml
person:
  name: 张三
  age: 18
  birthDay: 2010/10/10 12:12:12
  like: true
  child:
    name: 李四
    age: 20
    birthDay: 2018/10/10
    text: ["abc","def"]
  dogs:
    - name: 小黑
      age: 3
    - name: 小白
      age: 2
  cats:
    c1:
      name: 小蓝
      age: 3
    c2: {name: 小绿,age: 2} #对象也可用{}表示
```

### 日志模块

我们要知道，`springboot3`的日志分为 日志门面与日志实现

#### 日志门面 Facade

- **定义**：日志门面是一个抽象层，定义了一组标准的日志接口，允许开发者使用统一的日志 API，而无需关注底层使用的具体日志实现。常见的日志门面有 SLF4J（Simple Logging Facade for Java）和 JCL（Jakarta Commons Logging）。
- **作用**：日志门面通过提供一个统一的接口，让应用程序代码依赖于这个抽象层，而不是具体的日志实现库。这使得日志实现可以根据需求进行替换，而不需要修改应用程序代码。
- **在 Spring Boot 3 中**：Spring Boot 默认使用 SLF4J 作为日志门面。

####  日志实现

- **定义**：日志实现是具体的日志库，负责真正的日志输出。常见的日志实现包括 Logback、Log4j2 和 JUL（Java Util Logging）。
- **作用**：日志实现库负责将日志消息输出到控制台、文件或其他目的地。
- **在 Spring Boot 3 中**：默认情况下，Spring Boot 使用 Logback 作为日志实现。

默认使用Slf4j作为日志门面，我们在Java代码中使用`Slf4j`提供的 API进行开发，而底层SpringBoot默认使用`logback`作为默认底层实现，因此我们可以再不改变Javc层代码的同时，通过改变`Logback`的默认配置文件来实现日志定制化。我们在`Resource`目录中创建文件 `logback-spring.xml`，框架会默认读取。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 定义日志文件的存放路径 -->
    <property name="LOG_PATH" value="./logs" />
    
    <!-- 定义日志文件名的前缀 -->
    <property name="LOG_FILE" value="application" />

    <!-- 控制台输出的日志级别 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志输出的格式 -->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 滚动文件日志输出 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 日志文件的路径和名称 -->
        <file>${LOG_PATH}/${LOG_FILE}.log</file>
        
        <!-- 滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 定义日志文件的命名模式 -->
            <fileNamePattern>${LOG_PATH}/${LOG_FILE}-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 保留历史日志的天数 -->
            <maxHistory>30</maxHistory>
            <!-- 日志文件的大小上限 -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        
        <!-- 日志输出的格式 -->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 设置全局的日志级别，INFO、WARN、DEBUG、ERROR -->
    <root level="INFO">
        <!-- 将日志输出到控制台 -->
        <appender-ref ref="CONSOLE" />
        <!-- 将日志输出到文件 -->
        <appender-ref ref="FILE" />
    </root>

    <!-- 针对特定包或类的日志级别配置（可选） -->
    <logger name="com.example" level="DEBUG" additivity="false">
        <appender-ref ref="CONSOLE" />
    </logger>
</configuration>

```



#### 如何导入

1、每个`starter`场景，都会导入一个核心场景`spring-boot-starter`

2、核心场景引入了日志的所用功能`spring-boot-starter-logging`

3、默认使用了`logback + slf4j` 组合作为默认底层日志

4、`日志是系统一启动就要用`，`xxxAutoConfiguration`是系统启动好了以后放好的组件，后来用的，因此日志的配置**不是**使用 `自动配置`

5、日志是利用**监听器机制**配置好的。`ApplicationListener`。

6、日志所有的配置都可以通过修改配置文件实现。以`logging`开始的所有配置

#### 默认输出

```shell
2023-03-31T13:56:17.511+08:00  INFO 4944 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-03-31T13:56:17.511+08:00  INFO 4944 --- [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.7]
```



#### 使用

```java
Logger logger = LoggerFactory.getLogger(getClass());

// 或者使用Lombok的@Slf4j注解
```

**lombok注解实现：**

![@Slf4j.jpg](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/%40Slf4j.jpg) 

### 服务器

#### nginx

nginx是一个web服务器，可以用于部署web应用，其最大的两大特点就是：

1. 反向代理：一个非常好用的场景就是解决前端访问后端的跨域问题，即将前端的发送的动态请求由nginx转发到后端，对于前端来说，请求只需要发给自己就行了，不存在什么跨域。
2. 负载均衡：在反向代理的基础上，我们可以设定多个后端服务，用于nginx进行转发，nginx会使得前端发送的请求均匀的发送给部署个的各个后端！！！，淡然负载均衡nginx为我们提供了默认的轮询策略，当然我们也可以设定 权重、ip_hash、least_conn、url_hash等策略；
3. nginx默认使用了缓存机制，有时候并不需要真正访问后端服务器，同样的访问请求直接读取缓存就好了
4. 可以保证后端的安全：使用了反向代理之后后端服务器就可以简单的部署在公司内部局域网中，而不是公网当中！！！

![image-20240914150125425](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20240914150125425.png)

相关的配置都可以在nginx.conf文件中配置，

反向代理配置关键字为 proxy_pass，负载均衡的关在在于定义一个服务器地址集合，关键字为upstream：

```conf
upstream webservers{
	  server 127.0.0.1:8080 weight=90 ;
	  server 127.0.0.1:8088 weight=10 ;
}

# 反向代理,处理管理端发送的请求
location /api/ {
    proxy_pass   http://localhost:8080/admin/;
    #proxy_pass   http://webservers/admin/;
}

# 反向代理,处理用户端发送的请求(使用了负载均衡)
location /user/ {
    proxy_pass   http://webservers/user/;
}
```



#### Tomcat



### 好用工具类

Springboot为我们内置了很多好用的工具类例如：

https://blog.didispace.com/spring-boot-xxxUtils/

