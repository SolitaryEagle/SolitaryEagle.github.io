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
#### 合并复杂类型








