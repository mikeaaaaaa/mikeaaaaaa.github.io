---

title: 注解开发SpringBoot3
date: 2024-08-11 11:00:00 +0800
categories: [Springboot]
tags: [SpringBoot]

---
## 注解开发springboot3

### 总体概览

使用注解开发 `springboot3`，即不需要任何 `xml`文件，仅仅需要开发者配置好 各种属性配置文件 `*.properties`即可



### 	properties

上文说到使用了注解之后，我们只需要自己搞一些属性文件，那么问题来了，在`springboot3`框架中是如何使用到这些属性的呢？

#### 自动绑定机制

`springboot3`框架使用了自动绑定机制，即利用 `@ConfigurationProperties`注解以及 `@Value`注解，将`properties`文件中的属性自动与 `Java`对象绑定。

#### @ConfigurationProperties 注解

这个注解用于某一个 **组件类** 之上，类似于对这个类的所有属性都使用 `@Autowried`注解，知识自动注入的 源头来自于 `properties`文件，具体例子如下：

```java
package com.example.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "myapp.datasource")
public class DataSourceProperties {

    private String url;
    private String username;
    private String password;

    // Getters and Setters

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

之后，在需要的地方使用自动装配 `@Autowired`注解从容器中得到此组件使用即可。

#### @Value

`@Value`注解是Spring框架中的一个常用注解，用于为**类的字段**、**方法参数**或方法设置默认值，通常从外部配置文件（如`application.properties`、`application.yml`）或系统环境变量中注入值。这使得应用程序的配置更加灵活，可以根据不同的环境设置不同的配置参数。 **Spring会自动将配置中的字符串转换为目标类型。** 

```java
@Value("${my.property.name:DefaultValue}")
private String myProperty; // 如果my.property.name不存在，myProperty的值将是 "DefaultValue"
```

当然，这个注解也能用来引用系统或环境变量：

```java
@Value("${JAVA_HOME}")
private String javaHome;
```

方法参数中：

```java
@Component
public class MyService {

    private final String serviceUrl;

    public MyService(@Value("${service.url}") String serviceUrl) {
        this.serviceUrl = serviceUrl;
    }

    // 其他代码...
}

```



#### @PropertySource 

`@PropertySource`注解是将指定的属性文件加载到Spring的环境中，使得这些属性可以在整个Spring应用上下文中被访问和使用。 

```java
@Configuration
@PropertySource("classpath:application.properties")
@PropertySource("classpath:additional.properties")
public class AppConfig {
    // 配置类代码
}
```

#### 默认配置文件

Spring Boot 默认会在以下位置查找配置文件，并按优先级进行加载（从高到低）：

1. **外部配置文件**: 如 `file:./config/application.properties`
2. **项目根目录**: 如 `file:./application.properties`
3. **类路径**: 如 `classpath:/config/application.properties`
4. **默认配置文件**: 如 `classpath:/application.properties`

### 启动类

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.run(args);
    }
}
```

**@SpringBootApplication** 是一个复合注解，集成了以下三个注解：

- **@SpringBootConfiguration**：标识这是一个Spring Boot配置类，相当于一个Spring的`@Configuration`注解。
- **@EnableAutoConfiguration**：告诉Spring Boot自动配置Spring应用程序的上下文，尝试基于项目的依赖来自动配置Beans。
- **@ComponentScan**：启用组件扫描，以便Spring找到并注册标注了`@Component`、`@Service`、`@Repository`和`@Controller`等注解的Bean。

### Mybatis

**JDBC** 是Java中访问数据库的基础API，直接与数据库（如MySQL）通信。

**MyBatis** 是一个基于JDBC的持久层框架，它简化了数据库操作，提供了更高级的功能（如SQL映射、动态SQL等），但底层仍然使用JDBC来执行数据库操作。

**MySQL** 是一个关系型数据库管理系统，存储并管理应用程序中的数据。JDBC和MyBatis都是通过SQL语句与MySQL进行交互的。

#### 导入依赖

```xml
<!-- mybatis starter 开发场景 -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>3.0.0</version>
    </dependency>
```

当然，根据自动配置机制，我们还需要知道如何使用启动器：

![1724260055116](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/1724260055116.png)

官方`starter`文档：https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

官方文档中就讲解的很详细了：

| 属性                                     | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| `config-location`                        | **MyBatis 主配置文件的位置**<br>指定 MyBatis XML 配置文件（如 `mybatis-config.xml`）的路径，加载 MyBatis 的全局配置。 |
| `check-config-location`                  | **检查配置文件是否存在**<br>指示是否检查 `config-location` 所指定的 MyBatis XML 配置文件是否存在。 |
| `mapper-locations`                       | **Mapper XML 文件的位置**<br>指定 Mapper XML 文件的位置，支持多个路径，使用逗号、分号、制表符、换行符等分隔符分隔多个路径。 |
| `type-aliases-package`                   | **别名类型所在的包**<br>指定要扫描的别名类所在的包，MyBatis 会将包下的类自动注册为别名。包的路径可以用逗号、分号、制表符、换行符等分隔符分隔。 |
| `type-aliases-super-type`                | **别名类型的父类**<br>用于过滤 TypeAlias 的父类，只有此父类的子类才会被注册为别名。如果未指定，MyBatis 会将 `type-aliases-package` 中扫描到的所有类处理为 TypeAlias。 |
| `type-handlers-package`                  | **TypeHandler 类型所在的包**<br>指定要扫描的 TypeHandler 类所在的包，MyBatis 会自动注册这些 TypeHandler。包的路径可以用逗号、分号、制表符、换行符等分隔符分隔。 |
| `executor-type`                          | **执行器类型**<br>指定 MyBatis 的执行器类型，可选值为 `SIMPLE`（简单执行器）、`REUSE`（重用预处理语句）、`BATCH`（批处理）。 |
| `default-scripting-language-driver`      | **默认脚本语言驱动类**<br>指定默认的脚本语言驱动类，用于自定义 SQL 语句的执行方式。此功能需要与 mybatis-spring 2.0.2+ 一起使用。 |
| `configuration-properties`               | **外部化配置属性**<br>用于外部化 MyBatis 的配置属性，可以在 MyBatis 配置文件和 Mapper 文件中使用占位符引用这些属性。详细用法请参阅 MyBatis 官方文档。 |
| `lazy-initialization`                    | **Mapper Bean 懒加载**<br>指示是否启用 Mapper Bean 的懒加载。设置为 `true` 时，Mapper Bean 会在首次调用时初始化。此功能需要与 mybatis-spring 2.0.2+ 一起使用。 |
| `mapper-default-scope`                   | **Mapper Bean 的默认作用域**<br>指定自动配置扫描到的 Mapper Bean 的默认作用域，如 `singleton`（单例）。此功能需要与 mybatis-spring 2.0.6+ 一起使用。 |
| `inject-sql-session-on-mapper-scan`      | **Mapper 扫描时注入 SqlSession**<br>设置在 Mapper 扫描时是否注入 `SqlSessionTemplate` 或 `SqlSessionFactory` Bean。如果你希望恢复到 2.2.1 及之前的行为，设置为 `false`。在与 spring-native 一起使用时，建议设置为 `true`（默认）。 |
| `configuration.*`                        | **MyBatis 核心配置属性**<br>配置 MyBatis 核心的 `Configuration` Bean 属性，允许配置 MyBatis 核心的各种设置。注意：此属性不能与 `config-location` 同时使用。 |
| `scripting-language-driver.thymeleaf.*`  | **Thymeleaf 脚本语言驱动配置**<br>配置与 Thymeleaf 作为 MyBatis 脚本语言驱动相关的属性。详细用法请参阅 MyBatis Thymeleaf 官方文档。 |
| `scripting-language-driver.freemarker.*` | **FreeMarker 脚本语言驱动配置**<br>配置与 FreeMarker 作为 MyBatis 脚本语言驱动相关的属性。此功能需要与 mybatis-freemarker 1.2.0+ 一起使用。详细用法请参阅 MyBatis FreeMarker 官方文档。 |
| `scripting-language-driver.velocity.*`   | **Velocity 脚本语言驱动配置**<br>配置与 Velocity 作为 MyBatis 脚本语言驱动相关的属性。此功能需要与 mybatis-velocity 2.1.0+ 一起使用。详细用法请参阅 MyBatis Velocity 官方文档。 |





```properties
# MyBatis 核心配置文件路径
mybatis.config-location=classpath:/mybatis-config.xml

# MyBatis 映射文件路径，多个路径用逗号分隔
mybatis.mapper-locations=classpath:/mapper/*.xml,classpath:/another-mapper/*.xml

# 实体类别名所在的包,type-aliases-package 配置项指定了一个包路径，MyBatis 会自动扫描这个包中的所有类，并为每个类生成一个别名。默认情况下，别名是类的简单名称（不包括包名）。
mybatis.type-aliases-package=com.example.model

```



#### 配置类

在Spring Boot 3中，MyBatis的专属配置类可以配置多个方面，包括**数据源**、**SQL会话工厂**、**事务管理**、**Mapper扫描**、**MyBatis本身**的配置属性等。 

如果使用了myabtis-starter的话，其实下面这个配置类中的sqlsessionFactory组件也是可有可无的，

```java
package com.example.mybatisspringbootdemo.config;

// 说明本文件用来代替：mybatis-config.xml
@Configuration
// 配置扫描的mapper接口所在的包
@MapperScan("com.example.mybatisspringbootdemo.mapper")
public class MyBatisConfig {

    /**
     * 配置 SqlSessionFactory
     */
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath:mapper/*.xml"));
        return sessionFactoryBean.getObject();
    }
}

```

虽然使用注解也可以编写 `sql`语句，但是一旦难了之后就很难写，所以推荐直接使用 `mapper.xml`文件梭哈，下面我会介绍`mapper.ml`一些复杂的写法；

#### mapper

##### @Param

在XML中，`#{}`占位符会与`@Param`指定的参数名称对应。 如果不使用`@Param`，MyBatis会默认使用参数名称进行绑定 ；

本质：当使用`@Param`注解时，MyBatis会将参数封装到一个`Map`中，并使用注解中的名称作为键。 

```java
public interface UserMapper {
    User findUserByPriority(@Param("username") String username, @Param("email") String email);
}
```



##### 动态sql

MyBatis提供了动态SQL功能，通过在XML文件中使用标签和条件，可以根据传入的参数动态生成SQL语句。常用的标签包括`<if>`、`<choose>`、`<when>`、`<otherwise>`、`<trim>`、`<where>`、`<set>`等。 

###### \<if>表签

用于根据条件动态添加SQL片段 

```xml
<select id="findUsersByConditions" resultType="User">
    SELECT * FROM users
    <where>
        <if test="username != null">
            AND username = #{username}
        </if>
        <if test="email != null">
            AND email = #{email}
        </if>
        <if test="age != null">
            AND age = #{age}
        </if>
    </where>
</select>

```



###### \<choose>表签

用于实现`if-else if-else`的逻辑结构 ，第一个满足条件的`<when>`标签中的内容会被执行。如果没有条件满足，则执行`<otherwise>`标签中的内容 

```xml
<select id="findUserByPriority" resultType="User">
    SELECT * FROM users
    <where>
        <choose>
            <when test="username != null">
                AND username = #{username}
            </when>
            <when test="email != null">
                AND email = #{email}
            </when>
            <otherwise>
                AND status = 'active'
            </otherwise>
        </choose>
    </where>
</select>

```



###### \<trim> where set表签

- `<trim>`：用于自定义去除或添加SQL片段前后多余的内容，如前后的`AND`、`OR`。
- `<where>`：自动处理SQL中`WHERE`子句的开头`AND`或`OR`，并去除多余的条件。
- `<set>`：用于处理`UPDATE`语句中`SET`子句，自动去除前后多余的逗号。

```xml
xml
复制代码
<update id="updateUser">
    UPDATE users
    <set>
        <if test="username != null">
            username = #{username},
        </if>
        <if test="email != null">
            email = #{email},
        </if>
        <if test="age != null">
            age = #{age},
        </if>
    </set>
    WHERE id = #{id}
</update>
```

在这个`UPDATE`语句中，`<set>`标签会自动处理多余的逗号，使SQL更加干净。

##### 批量操作

###### 更新

```xml
<insert id="batchInsertUsers">
    INSERT INTO users (username, email, age)
    VALUES
    <foreach collection="userList" item="user" separator=",">
        (#{user.username}, #{user.email}, #{user.age})
    </foreach>
</insert>

```



###### 更新

```xml
<update id="batchUpdateUsers">
    <foreach collection="userList" item="user" separator=";">
        UPDATE users
        <set>
            <if test="user.username != null">
                username = #{user.username},
            </if>
            <if test="user.email != null">
                email = #{user.email},
            </if>
            <if test="user.age != null">
                age = #{user.age},
            </if>
        </set>
        WHERE id = #{user.id}
    </foreach>
</update>

```

##### 联合查询



```xml
<select id="findUsersWithOrders" resultType="User">
    SELECT u.*, o.order_id, o.order_date
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.username = #{username}
</select>

```

##### 子查询

```xml
<select id="findUsersWithLatestOrder" resultType="User">
    SELECT u.*, (SELECT MAX(order_date) FROM orders o WHERE o.user_id = u.id) as latest_order_date
    FROM users u
</select>

```

##### 自定义结果映射

当然一般的结果我们都尽量将其映射成一个对象，当然需要做的是 通过给sql语句数据库表字段加别名，让查询结果的每一列都和Java实体类中属性对应起来。 例如：

```sql
select emp_id empId,emp_name empName,emp_salary empSalary from t_emp where emp_id=#{maomi}
```

或者更加直接的：添加 `<setting name="mapUnderscoreToCamelCase" value="true"/> `配置，会自动 将Java中 `字母小写的驼峰式命名 `与sql中 `单词_单词 `字段进行自动匹配！！！

专门声明一个resultMap设定column到property之间的对应关系：



下面介绍复杂类型的映射：

`<association>`：用于处理多对一的关系，如User与Address的关系。

`<collection>`：用于处理一对多的关系，如User与Order的关系。

通常用于 联合查询的结果！！！

```xml
<resultMap id="UserOrderMap" type="User">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <result property="email" column="email"/>
    <collection property="orders" ofType="Order">
        <id property="orderId" column="order_id"/>
        <result property="orderDate" column="order_date"/>
    </collection>
</resultMap>

<select id="findUsersWithOrders" resultMap="UserOrderMap">
    SELECT u.id, u.username, u.email, o.order_id, o.order_date
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
</select>

```

##### map结果映射

MyBatis还可以将查询结果映射为一个`Map`对象，其中列名作为键，列值作为值。 适用于SQL查询返回的各个字段综合起来并不和任何一个现有的实体类对应，没法封装到实体类对象中。能够封装成实体类类型的，就不使用Map类型。 

```java
// Mapper接口
public interface UserMapper {
    Map<String, Object> findUserAsMap(Integer id);
}

```

```java
<!-- mapper.xml -->
<select id="findUserAsMap" resultType="map">
    SELECT * FROM users WHERE id = #{id}
</select>

```



##### 参数拆分

只要你在Mapper接口中传递的是一个Java对象（POJO），MyBatis就会自动拆分该对象的属性。你可以在SQL语句中直接引用这些属性，而无需额外的配置。 

如果你传递的是一个复杂对象，其中包含其他嵌套对象，MyBatis也能够处理这些嵌套属性。你可以通过`.`符号访问嵌套对象的属性，如`#{address.street}`

#### datasource 

`DataSource`为应用程序提供了一个统一的数据库连接管理方式，它可以自动处理连接的创建、复用、释放等操作，确保数据库操作的高效性和稳定性 。与传统的`DriverManager`不同，`DataSource`通常会配置连接池，这样可以复用；

##### HikariCP

`HikariCP`是一个非常快速且轻量级的JDBC连接池，Spring Boot默认使用它作为数据源的连接池实现。 



当Spring Boot应用启动时，自动配置模块会尝试根据项目中的依赖和配置文件（如`application.properties`或`application.yml`）中的信息，自动创建并配置 `DataSource`。这个过程由`DataSourceAutoConfiguration`类负责。 

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 连接池中允许的最大连接数
spring.datasource.hikari.maximum-pool-size=10
# 保持空闲状态的最小连接数
spring.datasource.hikari.minimum-idle=5
# 尝试获取连接时的最大等待时间，以毫秒为单位,系统将抛出一个SQL异常
spring.datasource.hikari.connection-timeout=20000

```

##### Druid

```xml
<dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>1.2.1</version>
            </dependency>
```

我们只需要在application中配置即可：

```yaml
spring:
  datasource:
    druid:
      driver-class-name: ${sky.datasource.driver-class-name}
      url: jdbc:mysql://${sky.datasource.host}:${sky.datasource.port}/${sky.datasource.database}?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
      username: ${sky.datasource.username}
      password: ${sky.datasource.password}
```



如果你需要完全控制`DataSource`的创建，可以自己定义一个`@Bean`方法来创建`DataSource`对象，这样Spring Boot的自动配置将不会生效。 

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;
import org.apache.tomcat.jdbc.pool.DataSource;

@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        DataSource dataSource = new DataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("secret");
        // additional settings
        return dataSource;
    }
}

```

### SpringMVC

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image.png)

**Servlet** 是 Java EE 规范的一部分，是用于处理 HTTP 请求并生成响应的 Java 类。Servlet 运行在 Servlet 容器（如 Apache Tomcat）中，Servlet 容器负责管理 Servlet 的生命周期，并将客户端请求传递给 Servlet 进行处理。**Servlet 容器** 提供了一套标准的接口，用于处理请求、管理会话、加载 Servlet 等。Servlet 主要用于处理低层次的 HTTP 请求和响应。 

#### SpringMVC原理：

Spring MVC 的核心组件 `DispatcherServlet` 就是一个标准的 Servlet。它继承自 `HttpServlet` 类，并且遵循 Servlet 规范。`DispatcherServlet` 作为前端控制器（Front Controller），拦截所有的 HTTP 请求，协调其他 Spring MVC 组件的工作，如控制器、视图解析器等。 Spring MVC 是基于 Servlet API 构建的。虽然 Spring MVC 提供了更高层次的抽象，但它依然依赖于底层的 Servlet 来处理请求和响应。直接使用Servlet进行开发需要大量的代码编写，而使用SpringMVC框架无疑提供了简洁高效的开发过程。同时虽然 Spring MVC 是基于 Servlet 的，但它提供了大量的扩展点（如拦截器、异常处理器、数据绑定等），使得开发者可以根据需要对请求处理流程进行定制。

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-1724401165758.png)

1. ##### DispatcherServlet

  作用: 这是SpringMVC的核心组件，也是前端控制器（Front Controller）。所有进入应用程序的HTTP请求首先会到达DispatcherServlet。它根据请求的URL和配置，确定如何处理请求。

  它是整个流程处理的核心，所有请求都经过它的处理和分发 

2. ##### HandlerMapping

  作用: 用于将请求URL映射到具体的处理器上（通常是Controller）。它负责根据请求路径选择合适的Controller。
  常用实现:
  RequestMappingHandlerMapping: 最常用的映射器，使用@RequestMapping注解来配置URL和Controller之间的映射关系。

3. ##### Controller

  作用: 处理HTTP请求的核心组件。Controller负责业务逻辑的处理，并将处理结果返回给DispatcherServlet。
  常用注解:
  @Controller: 标识一个类为Controller。
  @RequestMapping: 用于映射URL到Controller的方法上。

4. ##### Handler : handler

   又称处理器，他是Controller类内部的方法简称 是由我们自己定义，用来接收参数，向后调用业务，最终返回响应结果 

5. ##### Model

  作用: 用于在Controller与View之间传递数据。Model是一个数据容器，Controller将处理后的数据放入Model中，然后传递给View进行展示。
  常用接口:
  Model
  ModelMap
  ModelAndView: 可以同时携带Model数据和View信息。

6. ##### ViewResolver

  作用: 负责将逻辑视图名称（如"home"）解析为具体的视图实现（如JSP、Thymeleaf等）。ViewResolver根据配置，将Controller返回的视图名称转换为具体的视图对象。
  常用实现:
  InternalResourceViewResolver: 用于解析JSP视图。
  ThymeleafViewResolver: 用于解析Thymeleaf视图。

7. ##### View

  作用: 负责将Model中的数据渲染为用户可以看到的输出。View可以是各种形式，如JSP、HTML、JSON、XML等。
  常用实现:
  JstlView: 使用JSTL标签库的JSP视图。
  ThymeleafView: 使用Thymeleaf模板引擎的视图。
  MappingJackson2JsonView: 用于将Model数据渲染为JSON格式。

8. ##### HandlerAdapter

  作用: 负责调用Controller的方法处理请求。HandlerAdapter是DispatcherServlet和Controller之间的适配器，允许不同类型的处理器被调用。
  常用实现:
  RequestMappingHandlerAdapter: 处理带有@RequestMapping注解的方法。

9. ##### HandlerExceptionResolver

  作用: 处理在处理请求过程中抛出的异常。HandlerExceptionResolver可以根据异常类型或其它因素来决定如何处理异常并返回合适的视图。
  常用实现:
  ExceptionHandlerExceptionResolver: 通过@ExceptionHandler注解处理异常。
  DefaultHandlerExceptionResolver: 处理SpringMVC框架的标准异常。

10. ##### WebApplicationContext

  作用: WebApplicationContext是Spring容器的一种特殊实现，专门用于Web应用程序。它继承自ApplicationContext，并且集成了Web环境的特性，如Servlet API的访问等。它在SpringMVC应用中负责管理所有Spring Beans。
  组件之间的协作流程

  

  ##### 流程

  当用户发送一个HTTP请求时，流程通常是这样的：

+ 请求到达DispatcherServlet。
+ DispatcherServlet使用HandlerMapping确定哪个Controller处理请求。
+ 使用HandlerAdapter调用Controller中的相应方法。
+ Controller处理请求逻辑，可能会通过Model传递数据。
+ Controller返回视图名称给DispatcherServlet。
+ DispatcherServlet使用ViewResolver解析视图名称，获取具体的View对象。
+ DispatcherServlet将Model数据传给View进行渲染。

最终，View返回给客户端，展示给用户。 

#### 配置类

在开发时，通常需要对 Spring MVC 的配置进行自定义，如配置静态资源处理、视图解析器、跨域支持等。而具体我们如何定制我们需要的功能，方法就是实现`WebMvcConfigurer`抽象类提供的 `API`：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
public class WebJavaConfig implements WebMvcConfigurer {

    // 配置视图解析器
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        registry.viewResolver(resolver);
    }

    // 配置静态资源处理
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 配置静态资源路径，将 /resources/** 映射到 /static/ 目录
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("classpath:/static/");
    }

    // 开启静态资源的处理
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable(); 
    }

    // 配置跨域支持
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }

    // 其他自定义配置方法可以在此添加
}

```

#### 路经设置

##### 路劲匹配

**精确匹配**与**模糊匹配**

```java
@Controller
public class UserController {

    /**
     * 精准设置访问地址 /user/login
     */
    @RequestMapping(value = {"/user/login"})
    @ResponseBody
    public String login(){
        System.out.println("UserController.login");
        return "login success!!";
    }

    /**
     * 精准设置访问地址 /user/register
     */
    @RequestMapping(value = {"/user/register"})
    @ResponseBody
    public String register(){
        System.out.println("UserController.register");
        return "register success!!";
    }
    
    /**
     *  路径设置为 /product/*  
     *    /* 为单层任意字符串  /product/a  /product/aaa 可以访问此handler  
     *    /product/a/a 不可以
     *  路径设置为 /product/** 
     *   /** 为任意层任意字符串  /product/a  /product/aaa 可以访问此handler  
     *   /product/a/a 也可以访问
     */
    @RequestMapping("/product/*")
    @ResponseBody
    public String show(){
        System.out.println("ProductController.show");
        return "product show!";
    }
    
}

```

单层匹配和多层匹配： 

+ /*：只能匹配URL地址中的一层，如果想准确匹配两层，那么就写“/*/*”以此类推。

+ /**：可以匹配URL地址中的多层。 其中所谓的一层或多层是指一个URL地址字符串被“/”划分出来的各个层次 

  

  这个知识点虽然对于@RequestMapping注解来说实用性不大，但是将来配置拦截器的时候也遵循这个规则。 

##### @RequestMapping

`@RequestMapping` 注解可以用于类级别和方法级别，它们之间的区别如下：

1. 设置到类级别：`@RequestMapping` 注解可以设置在控制器类上，用于映射整个控制器的通用请求路径。这样，如果控制器中的多个方法都需要映射同一请求路径，就不需要在每个方法上都添加映射路径。
2. 设置到方法级别：`@RequestMapping` 注解也可以单独设置在控制器方法上，用于更细粒度地映射请求路径和处理方法。当多个方法处理同一个路径的不同操作时，可以使用方法级别的 `@RequestMapping` 注解进行更精细的映射。

```Java
//1.标记到handler方法
@RequestMapping("/user/login")
@RequestMapping("/user/register")
@RequestMapping("/user/logout")

//2.优化标记类+handler方法
//类上
@RequestMapping("/user")
//handler方法上
@RequestMapping("/login")
@RequestMapping("/register")
@RequestMapping("/logout")

```
`springmvc`默认有以下几种访问方法：

```java
public enum RequestMethod {
  GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE
}
```

使用`method`参数设置访问请求方法：

```java
@Controller
public class UserController {

    /**
     * 精准设置访问地址 /user/login，post
     * method = RequestMethod.POST 可以指定单个或者多个请求方式!
     * 注意:违背请求方式会出现405异常!
     */
    @RequestMapping(value = {"/user/login"} , method = RequestMethod.POST)
    @ResponseBody
    public String login(){
        System.out.println("UserController.login");
        return "login success!!";
    }

    /**
     * 精准设置访问地址 /user/register，get/post
     */
    @RequestMapping(value = {"/user/register"},method = {RequestMethod.POST,RequestMethod.GET})
    @ResponseBody
    public String register(){
        System.out.println("UserController.register");
        return "register success!!";
    }

}
```

##### 进阶注解

还有 `@RequestMapping` 的 HTTP 方法特定快捷方式变体：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

```Java
@RequestMapping(value="/login",method=RequestMethod.GET)
||
@GetMapping(value="/login")
```

注意：进阶注解只能添加到handler方法上，无法添加到类上！



#### 参数接收

param 类型的参数适用于单一的数据传递，而 JSON 类型的参数则更适用于更复杂的数据结构传递。根据具体的业务需求，需要选择合适的参数类型。在实际开发中，常见的做法是：在 GET 请求中采用 param 类型的参数，而在 POST 请求中采用 JSON 类型的参数传递。 

##### param参数接收

其实，只要方法参数与通过url传递的参数名相同，便可以自动接收

```java
@Controller
@RequestMapping("param")
public class ParamController {

    /**
     * 前端请求: http://localhost:8080/param/value?name=xx&age=18
     *
     * 可以利用形参列表,直接接收前端传递的param参数!
     *    要求: 参数名 = 形参名
     *          类型相同
     * 出现乱码正常，json接收具体解决！！
     * @return 返回前端数据
     */
    @GetMapping(value="/value")
    @ResponseBody
    public String setupForm(String name,int age){
        System.out.println("name = " + name + ", age = " + age);
        return name + age;
    }
}
```

当然，也可以使用 `@RequestParam`注解，可以将`Servlet`请求参数（查询参数或者表单数据）绑定到控制器中的方法参数上：

```java
 /**
 * 前端请求: http://localhost:8080/param/data?name=xx&stuAge=18
 * 
 *  使用@RequestParam注解标记handler方法的形参
 *  指定形参对应的请求参数@RequestParam(请求参数名称)
 */
@GetMapping(value="/data")
@ResponseBody
public Object paramForm(@RequestParam("name") String name, 
                        @RequestParam(value = "stuAge",required = false,defaultValue = "18") int age){
    System.out.println("name = " + name + ", age = " + age);
    return name+age;
}

```

当然，也有很多的特殊情况，例如多选框会有以下情况出现：

```java
  /**
   * 前端请求: http://localhost:8080/param/mul?hbs=吃&hbs=喝
   *
   *  一名多值,可以使用集合接收即可!但是需要使用@RequestParam注解指定
   */
  @GetMapping(value="/mul")
  @ResponseBody
  public Object mulForm(@RequestParam List<String> hbs){
      System.out.println("hbs = " + hbs);
      return hbs;
  }
```

##### 路径参数接收

路径传递参数是一种在 URL 路径中传递参数的方式。在 RESTful 的 Web 应用程序中，经常使用路径传递参数来表示资源的唯一标识符或更复杂的表示方式。而 Spring MVC 框架提供了 `@PathVariable` 注解来处理路径传递参数。

`@PathVariable` 注解允许将 URL 中的占位符映射到控制器方法中的参数。

例如，如果我们想将 `/user/{id}` 路径下的 `{id}` 映射到控制器方法的一个参数中，则可以使用 `@PathVariable` 注解来实现。

下面是一个使用 `@PathVariable` 注解处理路径传递参数的示例：

```java
 /**
 * 动态路径设计: /user/{动态部分}/{动态部分}   动态部分使用{}包含即可! {}内部动态标识!
 * 形参列表取值: @PathVariable Long id  如果形参名 = {动态标识} 自动赋值!
 *              @PathVariable("动态标识") Long id  如果形参名 != {动态标识} 可以通过指定动态标识赋值!
 *
 * 访问测试:  /param/user/1/root  -> id = 1  uname = root
 */
@GetMapping("/user/{id}/{name}")
@ResponseBody
public String getUser(@PathVariable Long id, 
                      @PathVariable("name") String uname) {
    System.out.println("id = " + id + ", uname = " + uname);
    return "user_detail";
}
```

##### json数据接收

前端传递 JSON 数据时，Spring MVC 框架可以使用 `@RequestBody` 注解来将 JSON 数据转换为 Java 对象。`@RequestBody` 注解表示当前方法参数的值应该从请求体中获取，并且需要指定 value 属性来指示请求体应该映射到哪个参数上。其使用方式和示例代码如下：

前端post方式发送的`json`数据如下以及对应的`java`类：

```java
{
  "name": "张三",
  "age": 18,
  "gender": "男"
}

public class Person {
  private String name;
  private int age;
  private String gender;
  // getter 和 setter 略
}
```

控制器中相应的方法如下：

```java
@PostMapping("/person")
@ResponseBody
public String addPerson(@RequestBody Person person) {

  // 在这里可以使用 person 对象来操作 JSON 数据中包含的属性
  return "success";
}
```

`@RequestBody` 注解将请求体中的 JSON 数据映射到 Person 类型的 person 参数上，并将其作为一个对象来传递给 addPerson() 方法进行处理 ；

`dependence`：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.0</version>
</dependency>
```

##### 接收Cookie参数

加入有一个请求带有如下的Cookie：

```python
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

使用 @CookieValue 将Cookie中的值作为控制器参数：

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
  //...
}
```

##### 接收请求头

假设有如下的请求头：

```xml
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

使用`@RequestHeader`，使请求头的值作为控制器方法的参数：

```java
@GetMapping("/post")
@ResponseBody
public parseHeader(@RequestHeader("Host") String host,@RequestHeader("Accept-Language") String acceptLanguage){
    
}
```

获取原生对象

|                                                              |                                                    |
| :----------------------------------------------------------: | :------------------------------------------------: |
| `jakarta.servlet.ServletRequest`, `jakarta.servlet.ServletResponse` |                   请求/响应对象                    |
|              `jakarta.servlet.http.HttpSession`              | 强制存在会话。因此，这样的参数永远不会为 `null` 。 |

使用也非常简单，直接作为控制器方法的形参传入即可，并且对顺序没有特别的要求，不分先后，再并且，其对其余参数的传入没有任何影响；

举例：

```java
/**
 * 如果想要获取请求或者响应对象,或者会话等,可以直接在形参列表传入,并且不分先后顺序!
 * 注意: 接收原生对象,并不影响参数接收!
 */
@GetMapping("api")
@ResponseBody
public String api(HttpSession session , HttpServletRequest request,
                  HttpServletResponse response){
    String method = request.getMethod();
    System.out.println("method = " + method);
    return "api";
}
```

#### 共享域

在 JavaWeb 中，共享域指的是在 Servlet 中存储数据，以便在同一 Web 应用程序的多个组件中进行共享和访问。常见的共享域有四种：`ServletContext`、`HttpSession`、`HttpServletRequest`、`PageContext`。

1. `ServletContext` 共享域：`ServletContext` 对象可以在整个 Web 应用程序中共享数据，是整个Web应用程序的上下文，代表了整个应用的范围。它在Web应用程序启动时创建，并在应用程序停止时销毁。`ServletContext` 对象在一个Web应用程序中是唯一的，所有Servlet和JSP都可以访问它。 适用于需要在整个应用程序中共享的数据，如应用程序级别的配置参数、全局计数器、数据库连接池等。 用法：

   ```java
   // 设置ServletContext范围内的属性
   getServletContext().setAttribute("appName", "MyWebApp");
   
   // 获取ServletContext范围内的属性
   String appName = (String) getServletContext().getAttribute("appName");
   ```

   

1. `HttpSession` 共享域：`HttpSession` 对象可以在同一用户发出的多个请求之间共享数据，但只能在同一个会话中使用。比如，可以将用户登录状态保存在 `HttpSession` 中，让用户在多个页面间保持登录状态。会话开始于用户第一次请求并创建 `HttpSession` 对象，结束于会话超时或显式无效化。

   用户登录状态的管理，如保存用户ID或用户名。

   存储购物车内容，在用户购买过程中保持会话的一致性。

   保存用户偏好设置或临时数据，方便用户在多个页面间操作。

   用法：

   ```java
   // 设置HttpSession范围内的属性
   HttpSession session = request.getSession();
   session.setAttribute("username", "JohnDoe");
   
   // 获取HttpSession范围内的属性
   String username = (String) session.getAttribute("username");
   ```

   

2. `HttpServletRequest` 共享域：`HttpServletRequest` 对象可以在同一个请求的多个处理器方法之间共享数据。基于单个请求的范围。每个HTTP请求都会创建一个新的 `HttpServletRequest` 对象，数据在该请求内共享，随着请求的处理完成而消失，如表单参数、请求头信息、用户输入数据等。 

   使用：

   ```java
   // 设置HttpServletRequest范围内的属性
   request.setAttribute("greeting", "Hello, World!");
   
   // 获取HttpServletRequest范围内的属性
   String greeting = (String) request.getAttribute("greeting");
   ```

   

3. `PageContext` 共享域：`PageContext` 对象是在 JSP 页面（是一种基于Java的Web技术，用于开发动态Web页面。它使得开发者可以在HTML代码中嵌入Java代码）Servlet 创建时自动创建的。它可以在 JSP 的各个作用域中共享数据，包括`pageScope`、`requestScope`、`sessionScope`、`applicationScope` 等作用域。

   使用：

   ```java
   // 设置PageContext范围内的属性
   pageContext.setAttribute("message", "Welcome to the page!");
   
   // 获取PageContext范围内的属性
   String message = (String) pageContext.getAttribute("message");
   ```

   

共享域的作用是提供了方便实用的方式在同一 Web 应用程序的多个组件之间传递数据，并且可以将数据保存在不同的共享域中，根据需要进行选择和使用。

![](https://secure2.wostatic.cn/static/wusZjB7KSU7gHUfXMpja5u/img010.png?auth_key=1724591930-wmZWr63SUjXbS5iJXvkQV1-0-4bd398f2aa4a43c20eb4590f48bfc53b)

在请求共享域共享数据

1. 使用原生 request 对象 ，request.setAttribute
2.  使用 Model 类型的形参 
3. 使用 ModelMap 类型的形参 
4. 使用 Map 类型的形参 
5. 使用 ModelAndView 对象 

#### 响应数据

##### jsp

JSP（JavaServer Pages）是一种动态网页开发技术，它是由 Sun 公司提出的一种基于 Java 技术的 Web 页面制作技术 ，我们在使用混合开发模式时，我们就需要使用到动态页面技术，动态展示Java的共享数据。

依赖：

```xml
<!-- jsp需要依赖! jstl-->
<dependency>
    <groupId>jakarta.servlet.jsp.jstl</groupId>
    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
    <version>3.0.0</version>
</dependency>
```

 jsp页面创建：建议位置 `/web-INF/views/home.jsp`

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>Title</title>
  </head>
  <body>
        <!-- 可以获取共享域的数据,动态展示! jsp== 后台vue -->
        ${msg}
  </body>
</html>

```

配置`jsp`视图解析器：

```java
@EnableWebMvc  //json数据处理,必须使用此注解,因为他会加入json处理器
@Configuration
@ComponentScan(basePackages = "com.atguigu.controller") //TODO: 进行controller扫描

//WebMvcConfigurer springMvc进行组件配置的规范,配置组件,提供各种方法! 前期可以实现
public class SpringMvcConfig implements WebMvcConfigurer {

    //配置jsp对应的视图解析器
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //快速配置jsp模板语言对应的
        registry.jsp("/WEB-INF/views/",".jsp");
    }
}
```

设置控制器函数 handler

```java
/**
 *  跳转到提交文件页面  /save/jump
 *  
 *  如果要返回jsp页面!
 *     1.方法返回值改成字符串类型
 *     2.返回逻辑视图名即可    
 *         <property name="prefix" value="/WEB-INF/views/"/>
 *            + 逻辑视图名 +
 *         <property name="suffix" value=".jsp"/>
 */
@GetMapping("jump")
public String jumpJsp(Model model){
    System.out.println("FileController.jumpJsp");
    model.addAttribute("msg","request data!!");
    return "home";
}
```

##### 转发与重定向

```java
@RequestMapping("/redirect-demo")
public String redirectDemo() {
    // 重定向到 /demo 路径 
    return "redirect:/demo";
}

@RequestMapping("/forward-demo")
public String forwardDemo() {
    // 转发到 /demo 路径
    return "forward:/demo";
}

//注意： 转发和重定向到项目下资源路径都是相同，都不需要添加项目根路径！填写项目下路径即可！
```

##### 返回json数据

添加依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.0</version>
</dependency>
```

`springmvc`自动为我们配置了 以 `jackson`作为`JSON`数据转换器，但是我们可以通过一下方式自定义数据转换器：在Spring MVC配置类（通常是使用`@Configuration`注解的类）中，可以通过重写`configureMessageConverters`或`extendMessageConverters`方法来添加或修改消息转换器。

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new MappingJackson2HttpMessageConverter());
    }
}
```

使用 `@RequestBody`以及 `@ResponseBody`注解：

**`@RequestBody`**: 将请求中的JSON数据反序列化为Java对象。

**`@ResponseBody`**: 将Java对象序列化为JSON数据并返回给客户端。

```java
@RequestMapping(value = "/user/detail", method = RequestMethod.POST)
@ResponseBody
public User getUser(@RequestBody User userParam) {
    System.out.println("userParam = " + userParam);
    User user = new User();
    user.setAge(18);
    user.setName("John");
    //返回的对象,会使用jackson的序列化工具,转成json返回给前端!
    return user;
}
```

另外 `Jackson`还支持非常多的注解用于 `Json`数据 与`java`对象之间的互相转化，常用的注解如下：

| 注解                    | 作用                                                         | 使用场景                                                     |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@JsonProperty`         | 指定序列化和反序列化过程中使用的JSON字段名称                 | 当Java字段名称和JSON字段名称不一致时使用                     |
| `@JsonIgnore`           | 在序列化和反序列化过程中忽略某个字段                         | 当你不希望某些字段出现在JSON中时使用                         |
| `@JsonIgnoreProperties` | 在类级别上指定要忽略的属性，可以指定多个                     | 当你想要忽略多个字段时使用                                   |
| `@JsonInclude`          | 控制哪些属性应该被包括在JSON中，可选择只包括非空、非默认值的属性 | 当你不希望在JSON中包括null值或默认值时使用                   |
| `@JsonFormat`           | 指定日期格式或其他格式化规则                                 | 当你需要格式化日期或其他类型的字段时使用                     |
| `@JsonCreator`          | 用于指定反序列化时的构造方法或工厂方法                       | 当需要使用特定构造函数或工厂方法来创建对象时使用             |
| `@JsonValue`            | 指定单个方法或字段的值作为整个对象的JSON表示                 | 当你希望某个对象在序列化时只输出一个字段的值，而不是整个对象时使用 |
| `@JsonDeserialize`      | 指定自定义的反序列化器                                       | 当默认的反序列化行为不满足需求时使用                         |
| `@JsonSerialize`        | 指定自定义的序列化器                                         | 当默认的序列化行为不满足需求时使用                           |
| `@JsonAnyGetter`        | 用于动态获取对象中的所有字段并将其作为JSON的键值对           | 当你需要处理一个动态结构的JSON时使用                         |
| `@JsonAnySetter`        | 用于动态地将JSON的键值对存储到Java对象的Map中                | 当你需要处理一个动态结构的JSON时使用                         |
| `@JsonTypeInfo`         | 用于处理多态对象的序列化和反序列化，帮助Jackson区分对象的具体类型 | 当你需要处理继承结构中的JSON时使用                           |
| `@JsonSubTypes`         | 与`@JsonTypeInfo`配合使用，指定多态类型的子类型              | 当你需要处理继承结构中的不同子类时使用                       |

类上的 @ResponseBody 注解可以和 @Controller 注解合并为 @RestController 注解。所以使用了 @RestController 注解就相当于给类中的每个方法都加了 @ResponseBody 注解。

RestController源码:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
 
  /**
   * The value may indicate a suggestion for a logical component name,
   * to be turned into a Spring bean in case of an autodetected component.
   * @return the suggested component name, if any (or empty String otherwise)
   * @since 4.0.1
   */
  @AliasFor(annotation = Controller.class)
  String value() default "";
 
}
```

##### 静态资源访问

首先，需要确保Spring MVC不会将所有请求都交给Controller处理，而是让一些请求直接指向静态资源。你可以通过配置`WebMvcConfigurer`来实现。

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}


```



当`configurer.enable();`被调用时，Spring MVC会将所有无法匹配到Controller的请求转发给Servlet容器的默认Servlet。例如，在Tomcat中，这个默认Servlet通常叫做`DefaultServlet`，它会根据请求的路径去查找对应的静态资源并返回给客户端。

- **静态资源处理**: 例如，请求`/static/css/style.css`，如果没有被任何Controller处理，它将由默认Servlet负责，从文件系统中获取资源并返回。
- **优点**: 启用默认Servlet处理可以避免开发者手动配置静态资源处理路径，简化了项目配置，并且让项目更容易管理。

#### 异常处理

一般使用全局异常捕获类进行异常处理，`@ControllerAdvice`注解用于定义全局的异常处理逻辑。它作用于整个应用程序中所有的控制器，提供了集中管理异常处理的方式。当然我们是不想返回一个视图而是简单的`Json`数据，我们使用注解 `@RestControllerAdvice`即可。通常将这个类放置在 controller或者exception包下。

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public String handleRuntimeException(RuntimeException ex, Model model) {
        model.addAttribute("errorMessage", ex.getMessage());
        return "error";  // 返回全局错误页面
    }

    @ExceptionHandler(Exception.class)
    public String handleException(Exception ex, Model model) {
        model.addAttribute("errorMessage", "An unexpected error occurred: " + ex.getMessage());
        return "genericError";  // 处理其他未捕获的异常
    }
}

```

#### 拦截器使用

说白了就是在请求到达 `handler`方法之前，先进行检查！

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-1724750372880-3-1724750383283-6.png)

创建拦截器类：

```java
public class Process01Interceptor implements HandlerInterceptor {

    // if( ! preHandler()){return;}
    // 在处理请求的目标 handler 方法前执行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("request = " + request + ", response = " + response + ", handler = " + handler);
        System.out.println("Process01Interceptor.preHandle");
         
        // 返回true：放行
        // 返回false：不放行
        return true;
    }
 
    // 在目标 handler 方法之后，handler报错不执行!
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("request = " + request + ", response = " + response + ", handler = " + handler + ", modelAndView = " + modelAndView);
        System.out.println("Process01Interceptor.postHandle");
    }
 
    // 渲染视图之后执行(最后),一定执行!
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("request = " + request + ", response = " + response + ", handler = " + handler + ", ex = " + ex);
        System.out.println("Process01Interceptor.afterCompletion");
    }
}
```



配置类添加拦截器：

```java
@EnableWebMvc  //json数据处理,必须使用此注解,因为他会加入json处理器
@Configuration
@ComponentScan(basePackages = {"com.atguigu.controller","com.atguigu.exceptionhandler"}) //TODO: 进行controller扫描
//WebMvcConfigurer springMvc进行组件配置的规范,配置组件,提供各种方法! 前期可以实现
public class SpringMvcConfig implements WebMvcConfigurer {

    //配置jsp对应的视图解析器
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        //快速配置jsp模板语言对应的
        registry.jsp("/WEB-INF/views/",".jsp");
    }

    //开启静态资源处理 <mvc:default-servlet-handler/>
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //添加拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) { 
        //将拦截器添加到Springmvc环境,默认拦截所有Springmvc分发的请求
        registry.addInterceptor(new Process01Interceptor());
    }
}


```

全面拦截与指定拦截：

```java
// 全面拦截
@Override
public void addInterceptors(InterceptorRegistry registry) {
    //将拦截器添加到Springmvc环境,默认拦截所有Springmvc分发的请求
    registry.addInterceptor(new Process01Interceptor());
}

// 精确拦截
@Override
public void addInterceptors(InterceptorRegistry registry) {
    
    //将拦截器添加到Springmvc环境,默认拦截所有Springmvc分发的请求
    registry.addInterceptor(new Process01Interceptor());
    
    //精准匹配,设置拦截器处理指定请求 路径可以设置一个或者多个,为项目下路径即可
    //addPathPatterns("/common/request/one") 添加拦截路径
    //也支持 /* 和 /** 模糊路径。 * 任意一层字符串 ** 任意层 任意字符串
    registry.addInterceptor(new Process01Interceptor()).addPathPatterns("/common/request/one","/common/request/tow");
}

```







拦截器与过滤器区别：

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/v2-4ccaed12613ae5fcb93bad94a80c4316_1440w-1724750412718-8.webp)

1、过滤器和拦截器**触发时机不一样**，**过滤器是**在请**求进入容器后**，但请求**进入servlet之前**进**行预处理**的。请求结束返回也是，是在servlet处理完后，返回给前端之前。

2、**拦截器**可以获取IOC容器中的各个bean，而过滤器就不行，因为拦**截器是spring提供并管理的**，spring的功能可以被拦截器使用，在拦截器里注入一个service，可以调用业务逻辑。而过滤器是JavaEE标准，只需依赖servlet api ，不需要依赖spring。

3、**过滤器的实现**基于**[回调函数](https://zhida.zhihu.com/search?q=回调函数&zhida_source=entity&is_preview=1)**。而**拦截器**（代理模式）的实现**基于反射**

4、**Filter**是依**赖于Servlet容**器，**属于Servlet规范的一部分**，而**拦截器则是独立存**在的，可以在任何情况下使用。

5、**Filte**r的执行由**Servlet容器回调完成**，而**拦截器**通常通**过动态代理（反射）**的方式来执行。

6、**Filter的生命周**期**由Servlet容器管理**，而**拦截器则**可以通过I**oC容器来管理**，因此可以通过注入等方式来获取其他Bean的实例，因此使用会更方便。

Servlet容器与Spring MVC框架之间的关系是层次性的，Spring MVC依赖于Servlet容器来运行。可以将Servlet容器看作是运行环境，而Spring MVC框架则是基于这个环境构建的Web应用框架。

Servlet容器与Spring MVC的关系

- **Servlet容器运行Spring MVC应用**: Spring MVC应用程序必须部署在一个Servlet容器中，如Tomcat、Jetty或GlassFish等。Servlet容器启动后，会加载并初始化Spring MVC的`DispatcherServlet`，并将所有请求交给它处理。
- **DispatcherServlet的工作原理**:
  1. **请求接收**: 当一个HTTP请求到达Servlet容器时，容器会根据配置（通常在`web.xml`或Spring Boot自动配置中）将请求交给`DispatcherServlet`处理。
  2. **请求处理**: `DispatcherServlet`根据请求的URL查找对应的控制器（Controller），然后调用相应的控制器方法处理请求。
  3. **视图渲染**: 控制器处理完请求后，返回一个视图名或数据对象。`DispatcherServlet`通过`ViewResolver`将视图名解析为实际的视图，并使用`ModelAndView`渲染视图，生成响应返回给客户端。
- **生命周期管理**: `DispatcherServlet`的生命周期由Servlet容器管理。容器负责在应用程序启动时初始化`DispatcherServlet`，在应用程序关闭时销毁它。

### SSM整合

#### 整体介绍

在学习SpringMVC与Mybatis后，我们就可以开始学习`SSM`了，其实`SSM`本质就是将三种技术做了整合，他们分别是 前面提及的两以及 提供注入、依赖管理、容器的`Spring`框架。

SSM框架一般需要两个容器,分别负责不同的职责：

1. **Root Application Context（根应用上下文）**:

   - **职责**: 管理Spring的核心Bean，如服务层（Service）、数据层（DAO）、事务管理器等。
   - **加载方式**: 通常在`web.xml`中通过`ContextLoaderListener`加载。

2. **Web Application Context（Web应用上下文）**:

   - **职责**: 管理Spring MVC的Bean，如控制器（Controller）、视图解析器等。
   - **加载方式**: 通常由`DispatcherServlet`加载，该容器是Root Application Context的子容器。

   |                   |                               |          |
   | ----------------- | ----------------------------- | -------- |
   | 配置名            | 对应内容                      | 对应容器 |
   | WebJavaConfig     | controller,springmvc相关      | web容器  |
   | ServiceJavaConfig | service,aop,tx相关            | root容器 |
   | MapperJavaConfig  | mapper,datasource,mybatis相关 | root容器 |

注：子容器可以单向的使用父容器中的组件进行注入！！！！

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-1725192036631-1.png)

#### 配置

##### 容器配置

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

  //指定root容器对应的配置类
  //root容器的配置类
  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class<?>[] { ServiceJavaConfig.class,MapperJavaConfig.class };
  }
  
  //指定web容器对应的配置类 webioc容器的配置类
  @Override
  protected Class<?>[] getServletConfigClasses() {
    return new Class<?>[] { WebJavaConfig.class };
  }
  
  //指定dispatcherServlet处理路径，通常为 / 
  @Override
  protected String[] getServletMappings() {
    return new String[] { "/" };
  }
}
```

 ##### WebJavaConfig

```java
/**
 * projectName: com.atguigu.config
 * 
 * 1.实现Springmvc组件声明标准化接口WebMvcConfigurer 提供了各种组件对应的方法
 * 2.添加配置类注解@Configuration
 * 3.添加mvc复合功能开关@EnableWebMvc
 * 4.添加controller层扫描注解
 * 5.开启默认处理器,支持静态资源处理
 */
@Configuration
@EnableWebMvc
@ComponentScan("com.atguigu.controller")
public class WebJavaConfig implements WebMvcConfigurer {

    //开启静态资源
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable(); 
    }
}

```

##### ServiceJavaConfig

```java
/**
 * projectName: com.atguigu.config
 * 
 * 1. 声明@Configuration注解,代表配置类
 * 2. 声明@EnableTransactionManagement注解,开启事务注解支持
 * 3. 声明@EnableAspectJAutoProxy注解,开启aspect aop注解支持
 * 4. 声明@ComponentScan("com.atguigu.service")注解,进行业务组件扫描
 * 5. 声明transactionManager(DataSource dataSource)方法,指定具体的事务管理器
 */
@EnableTransactionManagement
@EnableAspectJAutoProxy
@Configuration
@ComponentScan("com.atguigu.service")
public class ServiceJavaConfig {
    
    @Bean
    public DataSourceTransactionManager transactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
    
}
```

##### MapperJavaConfig

既不使用MybatisStarter，也不使用mybatis-xml配置文件，完全使用配置类！！！

![img](https://raw.githubusercontent.com/mikeaaaaaa/cloudimg/main/img/image-1725194360252-4.png)

不适用mybatis-starter版本：

```java
/**
 * projectName: com.atguigu.config
 *
 * description: 持久层配置和Druid和Mybatis配置 使用一个配置文件
 */
@Configuration
public class MapperJavaConfigNew {

    /**
     * 配置SqlSessionFactoryBean,指定连接池对象和外部配置文件即可
     * @param dataSource 需要注入连接池对象
     * @return 工厂Bean
     */
    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){
        //实例化SqlSessionFactory工厂
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();

        //设置连接池
        sqlSessionFactoryBean.setDataSource(dataSource);

        //settings [包裹到一个configuration对象,切记别倒错包]
        org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
        configuration.setMapUnderscoreToCamelCase(true);
        configuration.setLogImpl(Slf4jImpl.class);
        configuration.setAutoMappingBehavior(AutoMappingBehavior.FULL);
        sqlSessionFactoryBean.setConfiguration(configuration);

        //typeAliases
        sqlSessionFactoryBean.setTypeAliasesPackage("com.atguigu.pojo");

        //分页插件配置
        PageInterceptor pageInterceptor = new PageInterceptor();

        Properties properties = new Properties();
        properties.setProperty("helperDialect","mysql");
        pageInterceptor.setProperties(properties);
        sqlSessionFactoryBean.addPlugins(pageInterceptor);

        return sqlSessionFactoryBean;
    }

    /**
     * 配置Mapper实例扫描工厂,配置 <mapper <package 对应接口和mapperxml文件所在的包
     * @return
     */
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        //设置mapper接口和xml文件所在的共同包
        mapperScannerConfigurer.setBasePackage("com.atguigu.mapper");
        return mapperScannerConfigurer;
    }

}
```

使用mybatis-starter版本：

```java
// 说明本文件用来代替：mybatis-config.xml
@Configuration
// 配置扫描的mapper接口所在的包
@MapperScan("com.example.mybatisspringbootdemo.mapper")
public class MyBatisConfig {

    
}
```



在application中添加：

```xml
mybatis.mapper-locations=classpath:mapper/*.xml  // 指定mapper.xml文件的位置
```

其实，我们在使用mybatis-starter时，我们得看一下其原理，众所周知，含有`starter`的依赖一般是整合了`springboot`框架，就是自动为我们配置了一些组件，而mybatis-starter自动为我们配置了 `sqlSesssionFactory`，其原理如下：

```java
@Bean
    @ConditionalOnMissingBean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        if (this.properties.getConfiguration() == null || this.properties.getConfiguration().getVfsImpl() == null) {
            factory.setVfs(SpringBootVFS.class);
        }

        if (StringUtils.hasText(this.properties.getConfigLocation())) {
            factory.setConfigLocation(this.resourceLoader.getResource(this.properties.getConfigLocation()));
        }

        this.applyConfiguration(factory);
        if (this.properties.getConfigurationProperties() != null) {
            factory.setConfigurationProperties(this.properties.getConfigurationProperties());
        }

        if (!ObjectUtils.isEmpty(this.interceptors)) {
            factory.setPlugins(this.interceptors);
        }

        if (this.databaseIdProvider != null) {
            factory.setDatabaseIdProvider(this.databaseIdProvider);
        }

        if (StringUtils.hasLength(this.properties.getTypeAliasesPackage())) {
            factory.setTypeAliasesPackage(this.properties.getTypeAliasesPackage());
        }

        if (this.properties.getTypeAliasesSuperType() != null) {
            factory.setTypeAliasesSuperType(this.properties.getTypeAliasesSuperType());
        }

        if (StringUtils.hasLength(this.properties.getTypeHandlersPackage())) {
            factory.setTypeHandlersPackage(this.properties.getTypeHandlersPackage());
        }

        if (!ObjectUtils.isEmpty(this.typeHandlers)) {
            factory.setTypeHandlers(this.typeHandlers);
        }

        Resource[] mapperLocations = this.properties.resolveMapperLocations();
        if (!ObjectUtils.isEmpty(mapperLocations)) {
            factory.setMapperLocations(mapperLocations);
        }

        Set<String> factoryPropertyNames = (Set)Stream.of((new BeanWrapperImpl(SqlSessionFactoryBean.class)).getPropertyDescriptors()).map(FeatureDescriptor::getName).collect(Collectors.toSet());
        Class<? extends LanguageDriver> defaultLanguageDriver = this.properties.getDefaultScriptingLanguageDriver();
        if (factoryPropertyNames.contains("scriptingLanguageDrivers") && !ObjectUtils.isEmpty(this.languageDrivers)) {
            factory.setScriptingLanguageDrivers(this.languageDrivers);
            if (defaultLanguageDriver == null && this.languageDrivers.length == 1) {
                defaultLanguageDriver = this.languageDrivers[0].getClass();
            }
        }

        if (factoryPropertyNames.contains("defaultScriptingLanguageDriver")) {
            factory.setDefaultScriptingLanguageDriver(defaultLanguageDriver);
        }

        this.applySqlSessionFactoryBeanCustomizers(factory);
        return factory.getObject();
    }
```

这也就不难解释，我们不能自己实现`sqlSessionfactory`以及`sqlSessionfactoryBean`；







