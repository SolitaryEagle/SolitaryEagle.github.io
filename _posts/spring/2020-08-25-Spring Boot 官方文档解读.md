---
layout:     post
title:      Spring Boot 官方文档解读
subtitle:   Spring Boot 学习
date:       2020-08-25
author:     Eagle Cool
header-img: img/post-bg-ios9-web.jpg
catalog: 	true
tags:       Spring
---
# Spring Boot 的目标
* 为所有 Spring 开发者提供根本上更快且更广泛访问的入门经验。
* 开箱即用，但由于需求开始与默认值有所出入，因此很快就会摆脱困境。
* 提供一系列大型项目共有的非功能性功能（例如嵌入式服务器，安全性，指标，运行状况检查和外部化配置）。
* 完全不需要代码生成，也不需要XML配置。

# Spring Boot 的功能
## SpringApplication
### 启动失败分析报告
### 懒(延迟)初始化
### 自定义 Banner
### 自定义 SpringApplication
### 流式 Builder API
### Application 可用性
### Application 事件 和 监听器
* 事件发送的顺序
    * ApplicationStartingEvent：run 启动但未进行任何处理，META-INF/spring.factories 中的 listeners 和 initializers 可以监听到此事件
    * ApplicationEnvironmentPreparedEvent：context 知道使用的 Environment 被创建之后，context 创建之前
    * ApplicationContextInitializedEvent：ApplicationContext 已经准备好， ApplicationContextInitializers 已经被调用，但 bean definitions 被加载之前
    * ApplicationPreparedEvent：bean definitions 被加载之后，refresh 开始之前
    * WebServerInitializedEvent：WebServer 准备好之后
    * ContextRefreshedEvent：refreshed 完成之后
    * ApplicationStartedEvent：refreshed 完成之后， ApplicationRunner 和 CommandLineRunner 执行之前
    * AvailabilityChangeEvent：紧随 LivenessState.CORRECT 之后，以指示该应用程序被视为处于活动状态
    * ApplicationReadyEvent：ApplicationRunner 和 CommandLineRunner 执行之后
    * AvailabilityChangeEvent：紧随 ReadinessState.ACCEPTING_TRAFFIC 之后，以指示应用程序已准备就绪，可以处理请求
    * ApplicationFailedEvent：启动时发生异常
### Web Environment
* 如果 Spring MVC 出现；使用 AnnotationConfigServletWebServerApplicationContext
* 如果 Spring MVC 没出现，Spring WebFlux 出现；使用 AnnotationConfigReactiveWebServerApplicationContext
* 否则；使用 AnnotationConfigApplicationContext
### 访问 Application 参数
* 命令行参数被封装到 ApplicationArguments 中
* 也可以使用 @Value 访问
### 使用 ApplicationRunner 或 CommandLineRunner
* ApplicationRunner run 方法的参数是 ApplicationArguments
* CommandLineRunner run 方法的参数是 String... args
### Application 退出
### Admin 功能
## 外部化配置
属性获取的顺序
* 当使用 devtools 时，获取 $HOME/.config/spring-boot 目录下的配置
* @TestPropertySource 注解的配置
* @SpringBootTest 的 properties 属性的配置
* 命令行参数
* 嵌入在 环境变量， 系统属性， 命令行参数 和 JDNI java:comp/env/spring.application.json 中的 SPRING_APPLICATION_JSON 配置
* ServletConfig 的初始化参数
* ServletContext 的初始化参数
* 来自 java:comp/env 的 JDNI 属性
* Java 的系统属性
* OS 的环境变量
* random.\* 中的属性
* jar 包外的特定于 profile 的属性配置文件：application-{profile}.properties / application-{profile}.yml
* jar 包内的特定于 profile 的属性文件配置：application-{profile}.properties / application-{profile}.yml
* jar 包外的默认属性配置文件：application.properties / application.yml
* jar 包内的默认属性配置文件：application.properties / application.yml
* @PropertySource 引入的属性配置文件：__注意__，application context refresh 时才会添加这里的属性到 Environment 中，logging.\*，spring.main.\* 等属性会在 refresh 之前被读取 
* 通过 SpringApplication.setDefaultProperties 指定的默认属性
### 配置 Random 属性值
* integers
* longs
* uuids
* strings
### 访问 命令行属性
### Application 属性文件
spring.config.location + spring.config.additional-location 逆序排列就是文件搜索的路径
* 当前目录的 /config 子目录
* 当前目录
* classpath 的 /config 子目录
* classpath 根目录
### 特定于 Profile 的属性文件
* 可以使用 application-{profile}.properties 策略来定义。如果没有显示的指定 profiles，application-default.properties 将被加载。
* 无论 特定的Profile 位于保内还是包外，都会覆盖标准 application.properties 中的属性。
* 如果存在多个 profile 被指定，且存在相同的属性，最后的 profile 覆盖前面的前面的 profile 属性。
* 如果 spring.config.location 指定了属性文件，则其对应的的 profile 属性文件将不被考虑；请使用目录。
### 属性中的占位符
### 加密属性
* Spring Boot 不提供任何内建支持加密属性值，但是提供了一个必要的 hook 去修改 Environment 中的属性值。
* EnvironmentPostProcessor 允许你在 application 启动之前操作 Environment。
* 可以考虑 Spring Cloud Vault 项目
### 使用 YAML 取代 Properties
#### 加载 YAML
* YamlPropertiesFactoryBean 加载 YAML 为 Properties
* YamlMapFactoryBean 加载 YAML 为 Map
#### 在 Environment 中暴露 YAML 作为 Properties
#### 多 profile 的 YAML 文档
#### YAML 的缺点
* YAML 不能使用 @PropertySource 加载
* 推荐不要混合使用 profile-specific YAML 和 multiple YAML
### 类型安全的配置属性
#### JavaBean 属性绑定
#### 构造器绑定
* 不能将构造函数绑定与常规 Spring 机制创建的 bean 一起使用，如：@Component @Bean @Import
* 不建议将 java.util.Optional 和 @ConfigurationProperties 一起使用
#### 启用 @ConfigurationProperties 注解类型
#### 使用 @ConfigurationProperties 注解类型
#### 第三方配置
#### 松绑定
#### 绑定 Maps
#### 绑定来自环境变量
* 将 . 替换为 _
* 删除 -
* 转为全大写
#### 合并复合类型
多处配置 List 或 Map 类型的属性，只有优先级最高的一处生效，不会合并属性。
#### 属性转换
* ConversionService
* CustomEditorConfigurer
* Converters and @ConfigurationPropertiesBinding
##### 转换 Duration
* 一个正常的 long，代表 milliseconds 或使用 @DurationUnit 指定
* [使用 Duration 格式化一个标准的 ISO-8601](https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html#parse-java.lang.CharSequence-)
* 一种可读性更强的格式，其中值和单位耦合在一起（例如 10s 表示10秒）
    * `ns` for nanoseconds
    * `us` for microseconds
    * `ms` for milliseconds
    * `s` for seconds
    * `m` for minutes
    * `h` for hours
    * `d` for days
##### 转换 Period 
* 一个正常的 int，代表 days 或使用 @DurationUnit 指定
* [使用 Period 格式化一个标准的 ISO-8601](https://docs.oracle.com/javase/8/docs/api/java/time/Period.html#parse-java.lang.CharSequence-)
* 一种可读性更强的格式，其中值和单位耦合在一起（例如 1y3d 表示 1 年和 3 天）
    * `y` for years
    * `m` for months
    * `w` for weeks
    * `d` for days
##### 转换 DataSize
* 一个正常的 long，代表 字节 或使用 @DataSizeUnit 指定
* 一种可读性更强的格式，其中值和单位耦合在一起（例如 10MB 表示 10 兆字节）
    * `B` for bytes
    * `KB` for kilobytes
    * `MB` for megabytes
    * `GB` for gigabytes
    * `TB` for terabytes
#### @ConfigurationProperties 校验
使用 @Validated 注解 @ConfigurationProperties 注解过的类，会产生 javax.validation 校验

spring-boot-actuator 会暴露所有的 @ConfigurationProperties beans 在 /actuator/configprops

下面这段话如何理解？

The configuration properties validator is created very early 
in the application’s lifecycle, and declaring the @Bean method as static 
lets the bean be created without having to instantiate the @Configuration class. 
Doing so avoids any problems that may be caused by early instantiation.
#### @ConfigurationProperties vs @Value

Feature | @ConfigurationProperties | @Value
------- | ------------------------ | -------
Relaxed binding | Yes | Limited
Meta-data support | Yes | No
SpEL 表达式 | No | Yes

## Profile

### 添加 Active Profiles

当在多个属性源中添加 spring.profiles.active 时，会使用高优先级的属性源中的值，并不会合并它们的值。

使用 spring.profiles.include 表示添加 profile 到 spring.profiles.active 中。
**但是多个属性源中的 spring.profiles.include 会不会合并呢？**

### 以编程的方式设置 Profiles

* `SpringApplication.setAdditionalProfiles(...)`
* `ConfigurableEnvironment`

### 特定于 Profile 的配置文件

## 日志

### 日志格式化

* Date and Time：毫秒精度和容易排序。
* Log Level：ERROR, WARN, INFO, DEBUG, or TRACE。
* Process ID。
* `---` 分隔线：用以区分真实的日志信息。
* Thread name：使用中括号包装，在console上输出时可能会缩减。
* Logger name：通常是 source class name，经常缩写
* The log message。

### Console 输出

#### 颜色编码输出

### 文件输出

### Logback 扩展

必须使用 logback-spring.xml 或 logging.config

#### 特定于 Profile 的配置

* `<springProfile>`

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

#### Environment 属性

* `<springProperty>`

## 国际化

## JSON

### Jackson

## 开发 Web 应用程序

### The “Spring Web MVC Framework”

#### Spring MVC 自动配置

* ContentNegotiatingViewResolver 和 BeanNameViewResolver beans
* 支持服务静态资源，包括 WebJars
* 自动注册 Converter，GenericConverter 和 Formatter
* 支持 HttpMessageConverters
* 自动注册 MessageCodesResolver
* 支持静态的 index.html
* 支持自定义 Favicon
* 自动使用 ConfigurableWebBindingInitializer bean

如果你想保持 Spring Boot MVC 配置并且添加自己的 MVC 配置(interceptors, formatters, view controllers, and other features)，
你需要添加一个 @Configuration 注解的 WebMvcConfigurer 的子类，**但是不要使用 @EnableWebMvc**。

如果你想保持 Spring Boot MVC 配置并且添加自己的 RequestMappingHandlerMapping，RequestMappingHandlerAdapter，
ExceptionHandlerExceptionResolver；你需要声明一个 WebMvcRegistrations bean，并使用它去提供上述实例。

如果你想完全控制 Spring MVC，你需要使用 @EnableWebMvc 注解。

#### HttpMessageConverters

使用 Spring Boot’s HttpMessageConverters 去添加 Spring MVC‘s HttpMessageConverter。

#### 自定义 Json 序列化 和 反序列化

* 使用 Jackson 的 module 模型
* 使用 Spring Boot 的 @JsonComponent

#### MessageCodesResolver

http 错误码解析器

#### 静态内容

默认情况下，Spring Boot 通过下列文件夹提供静态内容（类路径下或 ServletContext 根路径）：
* /static
* /public
* /resources
* /META-INF/resources

#### 欢迎页

* index.html
* index template

#### 自定义 Favicon

在静态内容中寻找 favicon.ico

#### 路径匹配与内容协商

HTTP 客户端使用 `"Accept"` 请求头进行内容协商。

Spring Boot 默认禁用后缀名匹配，即 `"GET /projects/spring-boot.json"` 不匹配 
`@GetMapping("/projects/spring-boot")`。

当 HTTP 客户端没有使用 `"Accept"` 请求头进行内容协商时，可以使用查询参数代替。
`"GET /projects/spring-boot?format=json"` 匹配 `@GetMapping("/projects/spring-boot")`。

#### ConfigurableWebBindingInitializer

Spring MVC 使用 WebBindingInitializer 去初始化 WebDataBinder。如果你创建自己的
ConfigurableWebBindingInitializer bean，Spring Boot 自动配置它给 Spring MVC。

##### 模板引擎

#### 错误处理

### 支持嵌入式 Servlet 容器

#### Servlets, Filters, and listeners

##### 使用 Spring Beans 注册 Servlets, Filters, and Listeners 

* ServletRegistrationBean
* FilterRegistrationBean
* ServletListenerRegistrationBean

##### Servlet Context 初始化

嵌入式 Servlet 容器不直接执行 `javax.servlet.ServletContainerInitializer` 或
`org.springframework.web.WebApplicationInitializer`。这是一个有意的设计决策，
目的是降低在 war 中运行的第三方库可能破坏 Spring Boot 应用程序的风险。

如果你想执行 Servlet Context 初始化，你应该注册一个 `org.springframework.boot.web.servlet.ServletContextInitializer`.
它提供一个 onStartup 方法去访问 ServletContext，这可以简单的作为已经存在的 WebApplicationInitializer 的适配器。

#### 扫描 Servlets, Filters, and listeners

* @WebServlet
* @WebFilter
* @WebListener
* @ServletComponentScan

#### ServletWebServerApplicationContext

#### 自定义嵌入式 Servlet 容器

##### 编程式自定义

##### 直接自定义 ConfigurableServletWebServerFactory

## 优雅关机

## 使用 SQL Databases

### 配置一个 DataSource

#### 嵌入式 DataSource 支持

#### 连接线上 Database

DataSource 连接池的选择策略：
* HikariCP
* Tomcat DataSource 连接池
* Commons DBCP2 

#### 连接 JNDI DataSource

### 使用 JdbcTemplate

### JPA and Spring Data JPA





















