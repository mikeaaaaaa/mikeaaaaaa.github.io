---

title: 廖雪峰Java学习笔记
date: 2024-09-30 11:00:00 +0800
categories: [java,Spring,SpringNoot]
tags: [Java,Spring.SPringBoot]

---

# Spring Data

## SpringData介绍 

在java开发中一提到数据存储，我们首先会想到的就是 `Mybatis`，但是mybatis这个持久层框架仅仅支持`mysql`这种关系型数据库，而不支持 `redis`、mongoDB、`neo4j`之类的非关系型数据库，而`SpringData`解决了这种难点，使用这个框架可以支持各种类型的数据库操作；![redis对比其余数据库- kosamino - 博客园](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1010726-20161129001551849-1993796524.png)

对于开发者来说，面对各种类型的数据库，我们都需要学习相关的使用方法吗？在学习完Spring Data后，答案是 否；它针对不同的数据库类型提供了不同的模板操作类（简化开发，提升开发效率）

提到**模板操作类**，这里我们简单介绍一下：（XXXTemplate）

这些类通常是提供与某些技术集成的便捷工具类。这些类抽象了底层的复杂性，使得开发人员能够更简单、直观地与外部系统或服务进行交互。例如 `RestTemplate`提供了 GET、POST、PUT、DELETE 等标准 HTTP 请求方法的简单封装、`KafkaTemplate`用于与 Apache Kafka 进行消息生产的交互。

`XXXTemplate` 类在 Spring Boot 中通常是对某个外部系统或技术的封装，简化了与该系统的交互，并提供了更易用的 API。

这些模板类统一管理了**连接池**、**资源释放**、**事务处理**等，使得开发人员可以专注于业务逻辑，而不必处理底层的技术细节。



![image-20240930204038867](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20240930204038867.png)

主要使用的模块：（使用最为广泛的就是JPA）

![image-20240930204242731](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20240930204242731.png)

SpringData的主要特性：

![image-20240930204432354](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20240930204432354.png)

## JPA

JPA是什么？

JPA是Sun官方提供的一种**ORM规范**（object relation mapping），希望实现持久层的**天下归一**，因此我们使用了JPA可以随意切换数据库； 

Mybatis只能算作**半自动**的ORM框架，因为还是需要自己写sql语句。Mybatis其实就是对jdbc做了封装，在国内Mybatis比Hibernate更加流行的原因是国内一般涉及到的sql语句更加复杂，多则几百行一条sql语句，而hibernate对于复杂sql语句的编写是不如mybatis来的方便的。在业务相对比较简单的系统时毫无疑问选择Hibernate。

JPA提供了什么？

![image-20241001140214340](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241001140214340.png)

JDBC和JPA都是一组规范、一组接口，那么他们有什么区别呢？

答：JPA是JDBC的升级版，不同处在于，JDBC是由各个关系型数据库实现的（这也就是为什么我们在使用JDBC之前需要导入例如mysql驱动的原因，这个驱动就是数据库对于JDBC接口的实现）

JPA是由ORM框架（就是比一般的mybatis更牛逼的对象映射框架，例如hibernate）实现的，并且JDBC使用`sql`语句进行通信，JPA使用面向对象方式，通过ORM框架来生成SQL，进行操作。并且JPA在JDBC之上，需要依赖JDBC才能操作数据库。

![image-20241001135455040](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241001135455040.png)

有了JPA，我们不再需要使用英语与外国人交流，而是使用中文，再由例如hibernate之类的ORM对象映射框架来翻译成sql语句，再使用Jdbc与美国人交流！！！作为中国人，我们不再 需要学会英语（sql）了！！！并且JPA具有各个国家语言的翻译器，支持各种数据库语言；



JPA的实现

JPA的实现我们称之为：ORM框架，springboot默认使用`hibernate`作为 SpringData的实现；

当我们使用 JPA 后，我们使用的包就变成了 `jakarta.persistence`，而不是 `org.hibernate`这样的第三方 Jar包

### Hibernate

又学到几个快捷键：

ctrl + b  === CTRL + 左键

CTRL + sheft + b === ctrl + alt + 左键 	



下面介绍单独的`Hibernate`我们该如何使用（之后我们会统一使用JAP提供的接口来实现下面代码）

配置参数：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/springdata?useSSL=false&amp;serverTimezone=UTC</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">rootroot</property>

        <!-- 会在日志中记录sql，默认sql -->
        <property name="show_sql">true</property>
        <!-- 是否格式化sql，默认false -->
        <property name="format_sql">true</property>
        <!-- 表的生成策略，默认none不自动生成，update会自动更新表结构，create不管三七二十一直接创建 -->
        <property name="hbm2ddl.auto">update</property>
        <!-- 数据库方言（） -->
        <property name="dialect">org.hibernate.dialect.MySQL5InnoDBDialect</property>

        <mapping class="com.xiaohao.pojo.Customer"></mapping>
    </session-factory>
</hibernate-configuration>

```

测试脚本：

```java
public class HibernateTest {
    // 会话工厂,用于获取会话对象
    private SessionFactory sessionFactory;

    /**
     * 初始化,在测试方法执行之前执行
     */
    @Before
    public void before() {
        StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure().build();
        sessionFactory = new MetadataSources(registry).buildMetadata().buildSessionFactory();

    }

    @Test
    public void test_C() {

        try(Session session = sessionFactory.openSession()){
            // 开启事务
            Transaction transaction = session.beginTransaction();
            // 执行操作
            // 保存数据
            Customer customer = new Customer(1, "张三", "北京");
            session.save(customer);
            // 提交事务
            session.getTransaction().commit();
        }

    }

    @Test
    public void test_R() {

        try(Session session = sessionFactory.openSession()){
            // 开启事务
            Transaction transaction = session.beginTransaction();
            // 执行操作
            // 读取数据，读取数据有两种方式1、懒加载 load 2、立即加载 get，
            Customer customer = session.get(Customer.class, 1);
            System.out.println(customer);
            // 提交事务
            session.getTransaction().commit();
        }

    }

    @Test
    public void test_U() {

        try(Session session = sessionFactory.openSession()){
            // 开启事务
            Transaction transaction = session.beginTransaction();
            // 执行操作
            // 修改数据
            Customer customer = new Customer(1, "小红", "南京");
            // saveOrUpdate方法，如果主键存在则更新，如果主键不存在则插入
            session.saveOrUpdate(customer);
            // 提交事务
            session.getTransaction().commit();
        }

    }
}
```



使用JPA：

JPA还是约定大于配置，需要在`Resource`目录下，创建`META-INF`文件夹，并在文件夹中创建 `persistence.xml`文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             version="3.0"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd">

    <persistence-unit name="jpa-customer">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.xiaohao.pojo.Customer</class>
        <exclude-unlisted-classes>true</exclude-unlisted-classes>
        <properties>
            <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver" />
            <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/springdata?useSSL=false&amp;serverTimezone=UTC" />
            <property name="jakarta.persistence.jdbc.user" value="root" />
            <property name="jakarta.persistence.jdbc.password" value="rootroot" />
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5InnoDBDialect" />
            <!-- 会在日志中记录sql，默认sql -->
            <property name="show_sql" value="true" />
            <!-- 是否格式化sql，默认false -->
            <property name="format_sql" value="true"/>

        </properties>
    </persistence-unit>



</persistence>
```









### 使用介绍

![image-20241001205138167](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241001205138167.png)



用更加形象的一个图片：

![在这里插入图片描述](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1779d4e9bdc4bedd07751a2c43db1488.webp)

各种方法与状态的关系：（对象的四种状态）

+ 瞬时状态：就是一个普通的Java对象，和持久化上下文无关，数据库中也没有对应的数据
+ 托管状态：使用EntityManager进行find或者persist操作返回的对象即处于托管状态，此对象已处于持久化上下文中，因此任何对该实体的更新都会同步到数据库中，表现为，对JAP对象进行set，但是不save，数据库也能自动跟新
+ 游离状态：当事务提交后，处于托管状态的对象就变成了游离状态，此时不处于持久化上下文中，因此任何对该对象的操作都不会同步到数据库中
+ 删除状态：来当调用EntityManager对实体进行delete后，该实体就处于删除状态，其本质就是一个瞬时状态的对象

![image-20241001205639082](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241001205639082.png)

 



![image-20241001210420938](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241001210420938.png)



### SpringData JPA

SpringData JPA 是 spring提供的一套用于简化JPA开发的框架，其底层使用动态代理（AOP面向切面编程 ）对 JPA 的各个接口进行代理增强。



按照约定好的规则进行 **方法命名**去写dao层接口，就可以在不实现接口实现的情况下实现对数据库的访问和操作。 同时提供了很对除了 CRUD 之外的功能，如 分页、排序、发杂查询等。（Dao层只需要按照命名规则写接口，就自动拥有了CRUD功能，而不需要写具体的接口实现）

SpringData JPA 让我们脱离了 DAO层操作，基本上所有 CRUD都可以依赖它来实现，在实际中推荐使用 **SpringData JPA + ORM**（如hibernate）完成操作，这样在切换不同的ORM框架时就非常方便了，更加解耦。

Spring提供了一个关于 SpringData所有框架的统一依赖管理，我们只需在父项目中添加：

```xml
<dependencyManagement>
        <dependencies>
<!-- 这是一个非常好的依赖，其为我们直接统一管理了SpringData子项目的版本，这种书写方式是固定的 -->
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-bom</artifactId>
                <version>2024.0.4</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

子项目中直接如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
    </dependency>
</dependencies>
```

我们就可以直接愉快的使用了 ，

一共帮我们添加了如下的依赖：![image-20241001214713323](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241001214713323.png)

还需要添加例如junut、hibernate、mysql-connector-java、连接池（德鲁伊）依赖以及spring test 依赖：

```xml
		<dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
		<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>6.1.13</version>
        </dependency>
```

这里我们先讲解一下 xml 的配置思路，再介绍 configuration的配置思路：



在`AppConfig`中启用声明式事务管理，创建`DataSource`：

```java
@Configuration
@ComponentScan
@EnableTransactionManager
@PropertySource("jdbc.properties")
public class AppConfig{
    @Bean
    DataSource createDataSource(){
        
    }
}
```

使用Hibernate时，我们需要创建一个`LocalSessionFactoryBean`，并让它再自动创建一个`SessionFactory`。使用JPA也是类似的，我们也创建一个`LocalContainerEntityManagerFactoryBean`，并让它再自动创建一个`EntityManagerFactory`：

```java
@Bean
public LocalContainerEntiryManagerFactoryBean createEntitymanagerFactory(@AutoWired DataSource dtaSource){
    var emFactory  = new LocalCOn
}
```



# Spring开发（廖雪峰）

注意，不要与Springboot搞混淆，单纯的Spring项目是不会自动生成一个类似 `XXXApplication.java` 的文件，只有在Springboot中，通常会有一个包含 `main` 方法的启动类（通常命名为 `XXXApplication.java`），这个类上会有 `@SpringBootApplication` 注解。该类的作用是启动整个应用，自动扫描和加载所有的 Spring 组件和配置。

**单纯的Spring项目，需要我们创建 一个 `AppConfig.java`的类，如果是非web项目，还需要我们在这个类中编写一个 main方法，来创建容器并使用容器示例来获取各个组件**，并且没有自动扫描各个组件的功能，需要我们再AppConfig类之上使用 `@ComponentScan`注解从而识别其他组件。

例如下面代码：

**先执行 `main` 方法**，这是整个程序的入口，**在 `main` 方法中启动 Spring 容器**，然后容器会根据 `@ComponentScan` 或其他配置加载相应的组件。

```java
@Configuration
@ComponentScan("com.example")  // 指定组件扫描的包
public class Appconfig {

    public static void main(String[] args) {
        // 先执行 main 方法
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Appconfig.class);
        // Spring IOC 容器在这里启动，进行组件扫描
        MyService myService = context.getBean(MyService.class);
        myService.doSomething();
    }
}

```



## IOC容器

首先我们需要知道什么是容器：

容器是一种**为某种特定组件的运行提供必要支持的一个软件环境**。例如，Tomcat就是一个Servlet容器，它可以为Servlet的运行提供运行环境。类似Docker这样的软件也是一个容器，它提供了必要的Linux环境以便运行一个特定的Linux进程。当然，容器除了提供运行环境以外，还提供了丰富的封装好了的 API。



Spring的核心就是提供了一个**IoC容器**，它可以**管理所有轻量级的JavaBean组件**，提供的**底层服务**包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等。



### 原理

IoC：Inverse of Control：控制反转

**问题**：如果一个系统有大量的组件，其生命周期和相互之间的依赖关系如果由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。 因此需要一项技术来 统一创建组件并设置他们的依赖关系。

IoC解决这个问题，将传统的程序自己掌握的控制权交给了IoC容器，

### 使用

怎么得到这个容器？如下所示，Spring容器就是 `ApplicationContext`，他是一个接口，有很多实现类，这里我们选择`ClassPathXmlApplicationContext`，表示它会自动从`classpath`中查找指定的XML配置文件，**实现类决定了 容器加载配置的方式！！！**例如，使用注解类的话就这样得到容器：

`ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");`

` ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);`

### 定制Bean

Scope：

+ 默认为 Singleton，单例
+ Prototype，原型，表示每次注入都会新创建一个对象；需要额外使用 `@Scope`注解

+ List注入：`@Autowired List<Validator> validators;`会将容器中所有的`Validator`对象，都注入进来。、
+ 可选注入， @Autowired(required = false)
+ 第三方Bean：我们直接在方法中创建这个对象，然后返回它！！！暴力有效
+ 当定义多个同类型Bean时，会报错，因为默认名字是 类名的首字母消息，命名冲突！！，因此我们需要重命名！！，由于@Autowired 默认是按照类型进行注入的，有多个类型会报错，我们需要结合 `@Qualifier`注解指定Bean的名字



### 工厂创建

很多时候，可以通过**工厂模式创建对象**。**Spring也提供了工厂模式**，允许定义一个工厂，然后由工厂创建真正的Bean。

用工厂模式创建Bean需要实现`FactoryBean`接口。实现代码如下：

```java
@Component
public class ZoneIdFactoryBean implements FactoryBean<ZoneId> {

    String zone = "Z";

    @Override
    public ZoneId getObject() throws Exception {
        return ZoneId.of(zone);
    }

    @Override
    public Class<?> getObjectType() {
        return ZoneId.class;
    }
}
```

Spring工厂模式的机制如下：当一个组件类实现了 `FactoryBean<Object>`注解后，Spring当然会实例化这个组件，但紧接着调用 `getObject`方法来获取真正的组件!!!这种工厂组件为了与一般组件区分，我们通常命名： `XXXFactoryBean`。

！！！由于@Bean就可以创建第三方Bean，因此 @Bean其实就是工厂方法，所有 XXXFactoryBean 用的越来越少了，现在大都直接创建。

### 文件注入

在Java项目中，我们通常会使用到 配置文件以及 资源文件，因此在使用Spring容器时，我们也可以把文件注入进来，方便文件读取。

Spring为我们提供了一个 `org.springframework.core.io.Resource`类，它可以像String、int一样使用@`Value`注入；

使用：

```java
@Component
public class AppService {
    @Value("classpath:/logo.txt")
    private Resource resource;

    private String logo;

    @PostConstruct
    public void init() throws IOException {
        try (var reader = new BufferedReader(
                new InputStreamReader(resource.getInputStream(), StandardCharsets.UTF_8))) {
            this.logo = reader.lines().collect(Collectors.joining("\n"));
        }
    }
}
```

注入`Resource`最常用的方式是通过classpath，即类似`classpath:/logo.txt`表示在classpath中搜索`logo.txt`文件，然后，我们直接调用`Resource.getInputStream()`就可以获取到输入流，避免了自己搜索文件的代码。

### 条件注入

关键词为`Conditional`：

![springboot对条件接口Condition的扩展和使用_接口参数strcondition-CSDN博客](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/dc210c05b96790bc2b14b8964a21efca.png)

## AOP

AOP的实现有很多，最常用的就是 `AspectJ`

AOP支持：`org.springframework:spring-aspects:6.0.0`

AOP呢有非常多的名词：

- Aspect：切面，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；

- Joinpoint：连接点，即定义在应用程序流程的何处插入切面的执行；

- Pointcut：切入点，即一组连接点的集合；

- Advice：增强，指特定连接点上执行的动作；

- Introduction：引介，指为一个已有的Java对象动态地增加新的接口；

- Weaving：织入，指将切面整合到程序的执行流程中；

- Interceptor：拦截器，是一种实现增强的方式；

- Target Object：目标对象，即真正执行业务的核心逻辑对象；

- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用

    ![J7Hrh](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/J7Hrh.png)

我们简短总结一下：

1. 定义执行方法，并在方法上通过AspectJ的注解告诉Spring应该在何处、何时调用此方法
2. 标记`@Component`和`@Aspect`
3. 在`@Configuration`类上标注`@EnableAspectJAutoProxy`

使用：首先在启动类上 `@EnableAspectJAutoProxy`使用这个注解，这样Spring容器就会自动查找带有@Aspect注解的 组件然后根据每个方法特定的@Before、@Around等注解吧`AOP`注入到特定的`Bean`中。

我们通过一段示例来了解原理：

```java
@Aspect
@Component
public class LoggingAspect {
    // 在执行UserService的每个方法前执行:
    @Before("execution(public * com.itranswarp.learnjava.service.UserService.*(..))")
    public void doAccessCheck() {
        System.err.println("[Before] do access check...");
    }

    // 在执行MailService的每个方法前后执行:
    @Around("execution(public * com.itranswarp.learnjava.service.MailService.*(..))")
    public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {
        System.err.println("[Around] start " + pjp.getSignature());
        Object retVal = pjp.proceed(); // 指定目标方法，有确定名目标方法是否执行的权利
        System.err.println("[Around] done " + pjp.getSignature());
        return retVal;
    }
}
```





实际开发使用：

在实际开发使用中，最好是被代理的类能够清晰知道自己被代理了，例如 事务管理一样，使用 `@Transactional`注解表明自己被代理了，因此注解方式，是实现AOP最好的实现，实现方法呢，也很简单，分三步走：

1. 定义注解：

```java
@Target(METHOD)
@Retention(RUNTION)
public @interface MetricTime{
    String vlues;
}
```



2. 在被代理处标记注解：

```java
@Component
public class UserService {
    // 监控register()方法性能:
    @MetricTime("register")
    public User register(String email, String password, String name) {
        ...
    }
    ...
}
```



3. 定义@Aspect组件

```java
// 该类是一个切面类，负责监控带有 @MetricTime 注解的方法执行时间
@Aspect
@Component  // @Component 标注表示这是一个 Spring Bean，Spring 容器会管理它的生命周期
public class MetricAspect {

    /**
     * 环绕通知，拦截所有带有 @MetricTime 注解的方法。
     * 通过 ProceedingJoinPoint 对方法进行拦截和执行，MetricTime 参数包含注解的值。
     *
     * @param joinPoint    当前连接点，提供对目标方法的调用
     * @param metricTime   @MetricTime 注解，获取注解的值
     * @return 方法执行的返回值
     * @throws Throwable   方法执行过程中抛出的异常
     */
    @Around("@annotation(metricTime)")  // 通过 @Around 注解定义环绕通知，拦截所有带有 @MetricTime 注解的方法
    public Object metric(ProceedingJoinPoint joinPoint, MetricTime metricTime) throws Throwable {
        // 获取注解的值，通常用于标识监控的名称
        String name = metricTime.value();
        
        // 记录方法执行的开始时间
        long start = System.currentTimeMillis();
        
        try {
            // 执行被拦截的方法，并返回结果
            return joinPoint.proceed();
        } finally {
            // 计算方法执行时间
            long t = System.currentTimeMillis() - start;
            
            // 将方法执行时间写入日志或输出到控制台，方便性能监控
            System.err.println("[Metrics] " + name + ": " + t + "ms");
        }
    }
}

```

这里学到关于`finally`语句的使用注意事项：

1. 执行 `try` 块中的代码。

2. 如果遇到 `return` 语句，先暂存 `return` 的结果（如果有）。如果遇到异常抛出，有catch 执行catch，没有则执行finally代码

3. 执行 `finally` 块中的代码。

4. 在 `finally` 块执行完之后，再返回之前暂存的 `return` 结果。



一个坑的点，是关于AOP底层实现的，由于其底层是是使用 CGLIB 实现对实例对象的动态代理的，我们详细了解一下原理：

由于CGLIB是通过代理方式实现的，因此final类不能代理，并且final方法也不能被代理。

在使用 CGLIB 动态代理时，CGLIB 会**在运行时动态生成一个代理类**，该代理类是**被代理类的子类**，并且代理类会**重写被代理类的非 `final` 方法**。因此，生成的代理类对象**继承**了被代理类的所有特性，可以像被代理类一样使用，但会在方法调用时插入代理逻辑。

动态生成的代码如下，难点不在于生成后的代码，而是怎么动态生成这些代码：

```java
// CGLIB代理类具体实现
public class HelloConcrete$$EnhancerByCGLIB$$e3734e52
  extends HelloConcrete
  implements Factory
{
  ...
     // 拦截器，用来确定哪些方法需要intercept
  private MethodInterceptor CGLIB$CALLBACK_0; // ~~
  ...
  // 重写继承的方法
  public final String sayHello(String paramString)
  {
    ...
    MethodInterceptor tmp17_14 = CGLIB$CALLBACK_0;
    if (tmp17_14 != null) {
	  // 将请求转发给MethodInterceptor.intercept()方法。
      return (String)tmp17_14.intercept(this, 
              CGLIB$sayHello$0$Method, 
              new Object[] { paramString }, 
              CGLIB$sayHello$0$Proxy);
    }
    return super.sayHello(paramString);
  }
  ...
}
```





## 数据库访问

### JDBC

普通使用JDBC通常有以下步骤：

- 创建全局`DataSource`实例，表示数据库连接池；
- 在需要读写数据库的方法内部，按如下步骤访问数据库：
    - 从全局`DataSource`实例获取`Connection`实例；
    - 通过`Connection`实例创建`PreparedStatement`实例；
    - 执行SQL语句，如果是查询，则通过`ResultSet`读取结果集，如果是修改，则获得`int`结果。

+ 正确编写JDBC代码的关键是使用`try ... finally`释放资源，涉及到事务的代码需要正确提交或回滚事务。



在spring中使用JDBC就大不相同了，首先我们通过IoC容器创建并管理一个`DataSource`实例，然后，Spring提供了一个`JdbcTemplate`，可以方便地让我们操作JDBC，因此，通常情况下，我们会实例化一个`JdbcTemplate`：

```java
@Bean
    DataSource createDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(jdbcUrl);
        config.setUsername(jdbcUsername);
        config.setPassword(jdbcPassword);
        config.addDataSourceProperty("autoCommit", "true");
        config.addDataSourceProperty("connectionTimeout", "5");
        config.addDataSourceProperty("idleTimeout", "60");
        return new HikariDataSource(config);
    }

    @Bean
    JdbcTemplate createJdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
```

在执行查询或更新操作时，`JdbcTemplate` 底层会通过 `DataSource` 获取一个数据库连接。操作完成后，`JdbcTemplate` 会自动释放连接资源，这通常包括关闭 `Connection`、`Statement`、以及 `ResultSet`，

使用jdbc对数据库进行操作就非常方便了！！！！！！

并且jdbc还支持spring的声明式事务，即 `DataSourceTransactionManager`

### 事务管理

最初的事务管理要这样写，先操作再 commit，出错就rollback 

（Tx 是 Transaction的缩写）

```java
TransactionStatus tx = null;
try {
    // 开启事务:
    tx = txManager.getTransaction(new DefaultTransactionDefinition());
    // 相关JDBC操作:
    jdbcTemplate.update("...");
    jdbcTemplate.update("...");
    // 提交事务:
    txManager.commit(tx);
} catch (RuntimeException e) {
    // 回滚事务:
    txManager.rollback(tx);
    throw e;
}
```



使用Spring提供的就简单的多了：

首先自己定义一个 `PlatformTransactionManager`类：

```java
@Configuration
@ComponentScan
@PropertySource("jdbc.properties")
public class AppConfig {
    ...
    @Bean
    PlatformTransactionManager createTxManager(@Autowired DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

其次在启动类 加上 `@EnableTransactionManagement` 注解，用于启动**声明式任务**。

时候只需要对需要事务管理的Service层方法加上 `@Transactional`注解即可；

```java
@Component
public class UserService {
    // 此public方法自动具有事务支持:
    @Transactional
    public User register(String email, String password, String name) {
       ...
    }
}
```

底层原理基于 `AOP`代理，就是实现了自己的一个代理类如下：

可以看到，其实Spring底层关于事务的处理，最后也是通过 `jdbc`来实现的。

```java
public class UserService$$EnhancerBySpringCGLIB extends UserService {
    UserService target = ...
    PlatformTransactionManager txManager = ...

    public User register(String email, String password, String name) {
        TransactionStatus tx = null;
        try {
            tx = txManager.getTransaction(new DefaultTransactionDefinition());
            target.register(email, password, name);
            txManager.commit(tx);
        } catch (RuntimeException e) {
            txManager.rollback(tx);
            throw e;
        }
    }
    ...
}
```



说明：

1. 默认情况下，只要出现了 `RunTimeException`异常，就会回滚
2. 当然，我们也可以自己指定异常，这只需要在@Transactional注解中写明就行
3. **为了简化代码，我们强烈建议业务异常体系从`RuntimeException`派生，这样就不必声明任何特殊异常即可让Spring的声明式事务正常工作**

```java
public class BusinessException extends RuntimeException {
    ...
}
```



事务传播：

Spring的声明式事务为事务传播定义了几个级别，**默认传播级别就是REQUIRED**，它的意思是，如果当前没有事务，就创建一个新事务，如果当前有事务，就加入到当前事务中执行，即在一个事务中调用另一个事务，并不会导致创建新的事务。**默认的事务传播级别是`REQUIRED`，它满足绝大部分的需求**。

事务传播的实现机理就是  `ThreadLocal`  ：Spring总是把JDBC相关的`Connection`和`TransactionStatus`实例绑定到`ThreadLocal`。如果一个事务方法从`ThreadLocal`未取到事务，那么它会打开一个新的JDBC连接，同时开启一个新的事务，否则，它就直接使用从`ThreadLocal`获取的JDBC连接以及`TransactionStatus`，因此事务的传播只能在同一个线程中传播，当跨线程时就没有事务传播这一说法了。



### Hibernate

使用原始的 `JdbcTenplate` + `RowMapper`可以实现最基础的 ORM（object-Relation-mapping），而Hibernate就是更加自动化的ORM框架，但是Hibernate底层依然需要 Jdbc，因此我们我要引入：

+ spring-context
+ spring-orm
+ jakarta.annotation-api
+ jakarta.persistence-api
+ hibernate-core
+ HikariCP
+ hsqldb



为了使用Hibernate，我们需要定义`LocalSessionFactoryBean`：

```java
@Configuration
@ComponentScan
@EnableTransactionManagement
@PropertySource("jdbc.properties")
public class AppConfig {
    @Bean
    DataSource createDataSource() {
        ...
    }
    
    @Bean
    LocalSessionFactoryBean createSessionFactory(@Autowired DataSource dataSource) {
        var props = new Properties();
        props.setProperty("hibernate.hbm2ddl.auto", "update"); // 没有就自动创建数据库的表结构
        props.setProperty("hibernate.dialect", "org.hibernate.dialect.HSQLDialect"); // 指示Hibernate使用的数据库是HSQLDB
        props.setProperty("hibernate.show_sql", "true"); // 打印执行的SQL
        var sessionFactoryBean = new LocalSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        // 扫描指定的package获取所有entity class:
        sessionFactoryBean.setPackagesToScan("com.itranswarp.learnjava.entity");
        sessionFactoryBean.setHibernateProperties(props);
        return sessionFactoryBean;
    }
}
```

根据之前的学习，我们知道SessionFactoryBean就是创建的SessionFactory组件的组件，在`Hibernate`中，**session 是一个封装了 JDBC Connection的示例，而SessionFactory封装了 DataSource（数据库连接池）**，因此我们通过SessionFactory创建session就相当于从数据库连接池中获取一个连接。

Hibernate使用的时 `HQL`，是自己独有的数据库语言，当真正翻译成 数据库语言时，会根据 我们**设置的方言**（Dialect）来 生成特定的 优化数据库语言。



此外，我们还需要设定 `HiberanteTransactionManager`，这是配合使用声明式事务所必须的：

```java
@Bean
PlatformTransactionManager createTxManager(@Autowired SessionFactory sessionFactory) {
    return new HibernateTransactionManager(sessionFactory);
}
```



某一个实体类：

下面使用的所有注解都来自 `jakarta.persistence`,他是JPA规范的一部分。

```java
@Entity  // 将该类声明为一个 JPA 实体，与数据库中的表映射
@Table("user")  // 默认与user表绑定，如果不是这个表，可以使用这个注解
public class User {
    @Id  // 标识该字段为主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // 主键生成策略，使用数据库的自增字段
    @Column(nullable = false, updatable = false)  // 数据库中的列属性，不能为空且不能更新
    public Long getId() { ... }  // 获取用户的唯一标识 ID

    @Column(nullable = false, unique = true, length = 100)  // 列属性，不能为空，且值必须唯一，长度为 100
    public String getEmail() { ... }  // 获取用户的电子邮件地址

    @Column(nullable = false, length = 100)  // 列属性，不能为空，长度为 100
    public String getPassword() { ... }  // 获取用户的密码

    @Column(nullable = false, length = 100)  // 列属性，不能为空，长度为 100
    public String getName() { ... }  // 获取用户的姓名

    @Column(nullable = false, updatable = false)  // 列属性，不能为空，且在创建后不能修改
    public Long getCreatedAt() { ... }  // 获取用户创建时间的时间戳
}
```



具体增删改查：

```java
@Component
@Transactional
public class UserDao{
    @Autowired
    private SessionFactory sessionFactory;
    
    public User register(String email, String password, String name) {
        // 创建一个User对象:
        User user = new User();
        // 设置好各个属性:
        user.setEmail(email);
        user.setPassword(password);
        user.setName(name);
        // 不要设置id，因为使用了自增主键
        // 保存到数据库:
        sessionFactory.getCurrentSession().persist(user);
        // 现在已经自动获得了id:
        System.out.println(user.getId());
        return user;
    }
    
    public boolean deleteUser(Long id) {
        User user = sessionFactory.getCurrentSession().byId(User.class).load(id);
        if (user != null) {
            sessionFactory.getCurrentSession().remove(user);
            return true;
        }
        return false;
    }
    // 先select到相应的User，在更新
    public void updateUser(Long id, String name) {
        User user = sessionFactory.getCurrentSession().byId(User.class).load(id);
        user.setName(name);
        sessionFactory.getCurrentSession().merge(user);
    }
    // 使用Hql语句查询
    public List<User> select(String email,String password){
        List<User> list = sessionFactory.getCurrentSession()
            .createQuery("from User u where u.email = ?1 and u.password = ?2", User.class)
            .setParameter(1, email).setParameter(2, password)
            .list();
        retrun list;
    }
    
    
}
```

### JPA

即 `JAVA persistence API`，JPA就是 一套 ORM标准，Hibernate 就实现了这套标准，而当用户使用`JPA`时，引用的就是 `java.persistence`这个 ”标准包“，而不是 hibernate这样的第三方包；但是 因为 JPA只是一套接口，供用户使用，因此还需要选择具体的接口实现，。JPA 和 hibernate的关系 其实 就是 JDBC 和mysql之间的关系。

由于hibernate既提供了他自己的接口，也同时提供了对于JPA接口的实现，因此我们只需要导入普通的  `org.hibernate:hibernate-core`即可。



使用Hibernate时，我们需要创建一个`LocalSessionFactoryBean`，并让它再自动创建一个`SessionFactory`。使用JPA也是类似的，我们也创建一个`LocalContainerEntityManagerFactoryBean`，并让它再自动创建一个`EntityManagerFactory`：

```java
@Configuration
@ComponentScan
@EnableTransactionManagement
@PropertySource("jdbc.properties")
public class AppConfig {
    @Bean
    DataSource createDataSource() { ... }
    
    @Bean
    public LocalContainerEntityManagerFactoryBean createEntityManagerFactory(@Autowired DataSource dataSource) {
        var emFactory = new LocalContainerEntityManagerFactoryBean();
        // 注入DataSource:
        emFactory.setDataSource(dataSource);
        // 扫描指定的package获取所有entity class:
        emFactory.setPackagesToScan(AbstractEntity.class.getPackageName());
        // 使用Hibernate作为JPA实现:（这里选择hibernate最为jpa实现）
        emFactory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        // 其他配置项:
        var props = new Properties();
        props.setProperty("hibernate.hbm2ddl.auto", "update"); // 生产环境不要使用
        props.setProperty("hibernate.dialect", "org.hibernate.dialect.HSQLDialect");
        props.setProperty("hibernate.show_sql", "true");
        emFactory.setJpaProperties(props);
        return emFactory;
    }
    
    @Bean // 这里有不一样了，使用了另一个PlatFormTransactionManager的子类。
    PlatformTransactionManager createTxManager(@Autowired EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

除了配置的工厂组件不一样外，其余使用和上一节提到的完全相同，

使用：

```java
@Component
@Transactional
public class UserService {
    @PersistenceContext // 注意这里注入方式，不再是使用 @Autowired
    EntityManager em;
}
```







我们回顾一下JDBC、Hibernate和JPA提供的接口，实际上，它们的关系如下：

| JDBC       | Hibernate      | JPA                  | Mybatis           |
| ---------- | -------------- | -------------------- | ----------------- |
| DataSource | SessionFactory | EntityManagerFactory | SqlSessionFactory |
| Connection | Session        | EntityManager        | sqlSession        |

`SessionFactory`和`EntityManagerFactory`相当于`DataSource`，`Session`和`EntityManager`相当于`Connection`。每次需要访问数据库的时候，需要获取新的`Session`和`EntityManager`，用完后再关闭。



### Mybatis

mybatis是介于全自动例如hibernate以及全手动jdbc之间的，半自动ORM框架。

配置：需要DataSource数据库连接池；mybatis有一个好处就是可以直接使用spring提供的声明式事务，可以直接使用和JDBC一样的：

```java
@Configuration
@ComponentScan
@EnableTransactionManagement
@PropertySource("jdbc.properties")
public class AppConfig {
    @Bean
    DataSource createDataSource() { ... }
    @Bean
    PlatformTransactionManager createTxManager(@Autowired DataSource dataSource){
        return new DataSourceTransactionManager(dataSouce);
    }
    
    @Bean
    SqlSessionFactoryBean createSqlSessionFactoryBean(@Autowired DataSource dataSource) {
        var sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }
}
```

由于对Mybatis非常熟悉了，这里就不过多介绍了。



## Web开发

### SpringMVC

提到Spring之前我的第一反应就是 web开发！！！，但其实不是的，Spring 是一个**全面的 Java 企业级应用开发框架**，它为 Java 应用提供了很多基础功能，覆盖面远超 Web 开发：

Spring 框架的核心目标是为开发者提供一种**简化、解耦和模块化的方式**来开发各种类型的 Java 应用程序，特别是企业级应用。它的主要特性包括：

- **IoC（控制反转）**：管理对象的创建、生命周期和依赖关系。
- **AOP（面向切面编程）**：简化事务管理、安全性、日志记录等跨领域功能的实现。
- **事务管理**：统一的事务管理框架，支持 JDBC、JPA 等多种数据访问方式。

这些特性适用于各种类型的 Java 应用，而不仅仅局限于 Web 开发，它也涵盖了数据访问、事务管理、消息驱动开发、微服务、批处理等多个领域。



我们在spring框架中提及web开发主要指的是 SpringMVC；首先我们先来回顾一下java学习中的web开发相关知识：

主要有Servlet容器（主要有apache tomcat、jetty）以及标准的`servlet`组件：

+ Servlet：能处理HTTP请求并将HTTP响应返回
+ JSP：一种嵌套JAVA代码的HTML，将被编译为Servlet
+ Filter：能过滤URl以实现拦截功能
+ Listener：监听指定事件，

此外Servlet容器为每一个web应用程序自动创建一个唯一的`ServletContext`，这个实例就代表了web应用程序本身。

![images](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/images.png)

这里对比一下Tomcat与Nginx（在使用用途上就有很大区别）：

**Tomcat**：

- 适合用于 Java EE 应用开发和部署，特别是使用 Servlet、JSP、Spring MVC 等 Java 技术栈的应用。
- 用于处理需要生成动态内容的请求，例如用户登录、数据库查询等。

**Nginx**：

- 主要用于作为 **静态文件服务器** 和 **反向代理服务器**。
- 常用于将请求代理给后端应用服务器，例如代理 Java 应用（Tomcat）、Node.js 应用，或者其他后端服务。
- 也常用于实现 **负载均衡**，将请求分发给多个后端服务器，提升应用的可扩展性和可靠性。



下面正式进入SPpring MVC的学习：

![springmvc页面路径 springmvc目录结构_MVC](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/01055516_6609dbc4c3f6087585.png)



和Spring配置一样，我们编写正常的 `AppConfig.java`后，只需要加上 `@EnableWebMvc`注解即可开启SpringMVC，除了创建`DataSource`、`JdbcTemplate`、`PlatformTransactionManager`外，`AppConfig`需要额外创建几个用于Spring MVC的Bean：

+ `WebMvcConfigurer`：这个组件并不是必须的，但是我们能通过这个组件来详细配置一些规则，比如说下面 我们复写了addResourceHandlers方法，目的是让SpringMvc能自动处理/static下的静态文件。
+ `ViewResolver`：这是一个必须创建的组件，因为SpringMVC允许集成任何模板引擎，使用哪个模板引擎，我们就需要在这个组件中配置哪个模板引擎，下面组件中：`ViewResolver`通过指定`prefix`和`suffix`来确定如何查找View。上述配置使用Pebble引擎，指定模板文件存放在`/WEB-INF/templates/`目录下。

```java
@Bean
WebMvcConfigurer createWebMvcConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/static/**").addResourceLocations("/static/");
        }
    };
}

@Bean
ViewResolver createViewResolver(@Autowired ServletContext servletContext) {
    var engine = new PebbleEngine.Builder().autoEscaping(true)
            // cache:
            .cacheActive(false)
            // loader:
            .loader(new Servlet5Loader(servletContext))
            .build();
    var viewResolver = new PebbleViewResolver(engine);
    viewResolver.setPrefix("/WEB-INF/templates/");
    viewResolver.setSuffix("");
    return viewResolver;
}
```



在普通的Spring项目中，我们可以很容易的创建Spring容器的实例如下：

```java
public static void main(String[] args) {
    var context = new AnnotationConfigApplicationContext(AppConfig.class);
}
```

但是问题来了，现在是Web应用程序，而Web应用程序总是由Servlet容器创建，那么，Spring容器应该由谁创建？在什么时候创建？Spring容器中的Controller又是如何通过Servlet调用的？

答：1. 在`web.xml`中配置`DispatcherServlet`：

```xml
<?xml version="1.0"?>
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.itranswarp.learnjava.AppConfig</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

初始化参数`contextClass`指定使用注解配置的`AnnotationConfigWebApplicationContext`，配置文件的位置参数`contextConfigLocation`指向`AppConfig`的完整类名，最后，把这个Servlet映射到`/*`，即处理所有URL。

上述配置可以看作一个样板配置，有了这个配置，Servlet容器会首先初始化Spring MVC的`DispatcherServlet`，在`DispatcherServlet`启动时，它根据配置`AppConfig`创建了一个类型是`WebApplicationContext`的IoC容器，完成所有Bean的初始化，并将容器绑到`ServletContext`上。

因为`DispatcherServlet`持有IoC容器，能从IoC容器中获取所有`@Controller`的Bean，因此，`DispatcherServlet`接收到所有HTTP请求后，根据Controller方法配置的路径，就可以正确地把请求转发到指定方法，并根据返回的`ModelAndView`决定如何渲染页面。

（在没有SpringBoot的情况下，`web.xml` 文件用于配置 Web 应用程序的启动过程和相关的 Servlet 配置。当一个 Spring 项目包含 `web.xml` 文件时，**它会被 Servlet 容器**（例如 Tomcat、Jetty）**自动加载**。





最后，我们在`AppConfig`中通过`main()`方法启动嵌入式Tomcat：

```java
public static void main(String[] args) throws Exception {
    // 创建一个 Tomcat 实例
    Tomcat tomcat = new Tomcat();
    
    // 设置 Tomcat 的端口，默认端口为 8080，如果系统属性 'port' 设置了值，则使用该值
    tomcat.setPort(Integer.getInteger("port", 8080));

    // 获取 Tomcat 的连接器（这里没有实际配置连接器，但是会触发连接器的初始化）
    tomcat.getConnector();

    // 添加 Web 应用，设置 Web 应用的上下文路径为根路径（""），并指定应用的实际路径
    // 'src/main/webapp' 是项目的 Web 应用目录
    Context ctx = tomcat.addWebapp("", new File("src/main/webapp").getAbsolutePath());

    // 创建资源的根目录，并将其关联到上下文中
    WebResourceRoot resources = new StandardRoot(ctx);

    // 将 'target/classes' 目录中的资源添加到 Web 应用的资源中，用于提供 /WEB-INF/classes 目录下的类和资源
    resources.addPreResources(
        new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));

    // 设置应用的资源为刚刚创建的资源
    ctx.setResources(resources);

    // 启动 Tomcat 服务器
    tomcat.start();

    // 让 Tomcat 服务器保持运行，等待请求
    tomcat.getServer().await();
}

```

上述代码中没有实际初始化IOC容器，因为Servlet容器已经自动加载web.xml文件创建过了。



对于Controller中，我们可以定义一个个方法，这些方法称之为**控制器方法**（在python中称之为路由方法）

这里我其实有个困惑的点，`handler`是什么？

答：在 Spring MVC 中，**Handler**（处理器）是指处理 HTTP 请求的核心组件。具体来说，Handler 负责根据客户端的请求做出响应，而一个 **Handler** 通常是一个带有注解的 **控制器方法**。控制器方法就是handler。

#### Convert

这里提一嘴SpringMVC强大的参数处理功能：

自定义的 `Converter` 可以用来处理各种自定义类型的转换需求，当然SpringMVC为我们实现了很多默认的`Converter`，他们有以下作用：

1. 基本数据类型与其**包装类**的自动转换
2. 提供对于 Date、LocalDate、LocalTime、LocalDateTime等**日期类型**自动转换的功能
3. 还可以将请求中的字符串转换为对应的**枚举类型**
4. 支持以**逗号**为分隔符字符串转换为**集合类型**（List、Set、Map）
5. 文件数据转换为 `MultipartFile` 类型
6. 路径变量自动转换为参数
7. 常用对象类型（如 `BigDecimal`、`UUID`）的转换
8. 当导入jackson时，还支持各种复杂对象的转换



#### 接收请求参数所有方式总结

这里再提一句SpringMVC对于 表单参数、Query参数、Post请求体参数处理的区别（请求参数所有类型）： 

**表单参数**：post方法，并且ContentType指定为`application/x-www-form-urlencoded`，

```java
@PostMapping(value = "/post")public String post(String name,  
                   Integer age) {  
    String content = String.format("name = %s,age = %d", name, age);  
    log.info(content);  
    return content;  
}  
```

当然，只要我们导入`jackson-databind`，我们就可以直接使用对象类进行接收：

```java
@PostMapping(value = "/post")public String post(User user) {  
    String content = String.format("name = %s,age = %d", name, age);  
    log.info(content);  
    return content;  
}  
```

**Json参数**：

导入`jackson-databind`后，直接RequestBody注解

**路径参数**：

`@PathVariable`

**query参数**：

在导入`jackson`后与表单参数如出一辙，都有两种方式可以读取。

![image-20241003171647177](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241003171647177.png)

**文件参数**：

```java
@PostMapping(value = "/file1")  
public String file1(MultipartFile image) {  
    String content = String.format("name = %s,originName = %s,size = %d",  
            multipartFile.getName(), multipartFile.getOriginalFilename(), multipartFile.getSize());  
    log.info(content);  
    return content;  
}  
```





### RESTful

在Web应用中，除了需要使用MVC给用户显示页面外，还有一类API接口，我们称之为REST，通常输入输出都是JSON，便于第三方调用或者使用页面JavaScript与之交互。

只需要 `@ResponseBody`  、`@RequestBody`这里注解或者使用 @`RestController`。

有个特殊的就是得添加`jackson`依赖，用于对象与JSON的转换：

> com.fasterxml.jackson.core:jackson-databind:2.14.0





#### Interceptor 与 Filter

Filter属于标准Servlet组件之一，因此是由Servlet管理，Filter组件实际上并不知道后续内部处理是通过Spring MVC提供的`DispatcherServlet`还是其他Servlet组件，因为Filter是Servlet规范定义的标准组件，它可以应用在任何基于Servlet的程序中。

因此创建它稍微有一点麻烦，因为它得由Servlet创建，但是Spring框架创建管理统一由IOC容器来做，因此这其中就有点麻烦，当然Spring也为我们提供了 `DelegatingFilterProxy`来专门干这件事：

![image-20241003173125236](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241003173125236.png)



那我们再来谈谈 `Interceptor`（**拦截器**）：

既然使用Filter这么麻烦，那我们为什么不脱离 Servlet 在SpringMVC中实现一个功能类似Filter的东西呢：`Interceptor`；因此这两者处理流程如下：![image-20241003173532052](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-20241003173532052.png)

所以，Interceptor的拦截范围其实就是Controller方法，它实际上就相当于基于AOP的方法拦截。因为Interceptor只拦截Controller方法，所以要注意，返回`ModelAndView`并渲染后，后续处理就脱离了Interceptor的拦截范围。

使用Interceptor的好处是Interceptor本身是Spring管理的Bean，因此注入任意Bean都非常简单（使用@Autowired 即可实现组件装配）。此外，可以应用多个Interceptor，并通过简单的`@Order`指定顺序。

使用方法：

一个Interceptor必须实现`HandlerInterceptor`接口，可以选择实现`preHandle()`、`postHandle()`和`afterCompletion()`方法。`preHandle()`是Controller方法调用前执行，`postHandle()`是Controller方法正常返回后执行，而`afterCompletion()`无论Controller方法是否抛异常都会执行，参数`ex`就是Controller方法抛出的异常（未抛出异常是`null`）。

在`preHandle()`中，也可以直接处理响应，然后返回`false`表示无需调用Controller方法继续处理了，通常在认证或者安全检查失败时直接返回错误响应。在`postHandle()`中，因为捕获了Controller方法返回的`ModelAndView`，所以可以继续往`ModelAndView`里添加一些通用数据，很多页面需要的全局数据如Copyright信息等都可以放到这里，无需在每个Controller方法中重复添加。

```java
@Order(2)
@Component
public class AuthInterceptor implements HandlerInterceptor {

    final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    UserService userService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        logger.info("pre authenticate {}...", request.getRequestURI());
        try {
            authenticateByHeader(request);
        } catch (RuntimeException e) {
            logger.warn("login by authorization header failed.", e);
        }
        return true;
    }

    private void authenticateByHeader(HttpServletRequest req) {
        String authHeader = req.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Basic ")) {
            logger.info("try authenticate by authorization header...");
            String up = new String(Base64.getDecoder().decode(authHeader.substring(6)), StandardCharsets.UTF_8);
            int pos = up.indexOf(':');
            if (pos > 0) {
                String email = URLDecoder.decode(up.substring(0, pos), StandardCharsets.UTF_8);
                String password = URLDecoder.decode(up.substring(pos + 1), StandardCharsets.UTF_8);
                User user = userService.signin(email, password);
                req.getSession().setAttribute(UserController.KEY_USER, user);
                logger.info("user {} login by authorization header ok.", email);
            }
        }
    }
}
这个AuthInterceptor是由Spring容器直接管理的，因此注入UserService非常方便。

最后，要让拦截器生效，我们在WebMvcConfigurer中注册所有的Interceptor：

@Bean
WebMvcConfigurer createWebMvcConfigurer(@Autowired HandlerInterceptor[] interceptors) {
    return new WebMvcConfigurer() {
        public void addInterceptors(InterceptorRegistry registry) {
            for (var interceptor : interceptors) {
                registry.addInterceptor(interceptor);
            }
        }
        ...
    };
}
```

在定义完Intercept后，我们还需要在`WebMvcConfigurer`中进行注册，确定拦截器生效范围：

```java
@Configuration
@Slf4j
public class WebMvcConfiguration extends WebMvcConfigurationSupport {
    protected void addInterceptors(InterceptorRegistry registry) {
            // 将自定义的拦截器注册到 Spring MVC 中
            log.info("开始注册自定义拦截器...");
            registry.addInterceptor(jwtTokenAdminInterceptor)
                    // 指定拦截器要作用的 URL 路径
                    .addPathPatterns("/admin/**")
                    .excludePathPatterns("/admin/employee/login");
            registry.addInterceptor(jwtTokenUserInterceptor)
                    // 指定拦截器要作用的 URL 路径
                    .addPathPatterns("/user/**")
                    .excludePathPatterns("/user/shop/status")
                    .excludePathPatterns("/user/user/login");
        }
}
```





## SpringBoot开发

我们先来看一下 web 框架演变流程：

![浅谈JavaWeb架构演变- Alan_beijing - 博客园](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1066923-20190215134419800-1367307662.png)











