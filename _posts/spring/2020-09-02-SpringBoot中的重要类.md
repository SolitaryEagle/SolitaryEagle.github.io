---
layout:     post
title:      SpringBoot 中的重要类
subtitle:   SpringBoot 中的重要类
date:       2020-09-02
author:     Eagle Cool
header-img: img/post-bg-ios9-web.jpg
catalog: 	true
tags:       Spring
---

* ApplicationContextInitializer


* ApplicationListener
* GenericApplicationListener
* SmartApplicationListener
* SpringApplicationRunListener
* EventPublishingRunListener
* ApplicationEventMulticaster
* SpringApplicationEvent


* ResolvableType


* LoggingSystem
* LoggerContext


* ApplicationArguments
* PropertySource
* MutablePropertySources
* EnumerablePropertySource
* CommandLinePropertySource
* SimpleCommandLineArgsParser
* PropertySourcesPropertyResolver
* PropertySourcesPlaceholdersResolver
* PropertySourcesPlaceholderConfigurer
* PropertyPlaceholderHelper
* ConfigurableEnvironment
* ConfigurableWebEnvironment
* ConfigurablePropertyResolver
* PropertySourceLoader
* OriginTrackedYamlLoader
* Binder
* ConfigurationPropertySource
* PropertyMapper
* ConfigurationProperty
* ConfigurationPropertyName


* ApplicationConversionService
* GenericConverter
* TypeDescriptor
* ConvertiblePair


* ApplicationContext
* BeanFactoryPostProcessor
* BeanPostProcessor
* BeanWrapperImpl
* Converter

* EnvironmentPostProcessor 


* JsonParserFactory
* JsonParser


* ParameterNameDiscoverer


* RootBeanDefinition
* AnnotationTypeFilter

# 待做

SpringApplication.setDefaultProperties();
applyActiveProfiles(defaultProperties);

Spring 属性绑定 vs SpringBoot 属性绑定

org.springframework.boot.SpringApplication#lazyInitialization 设置为 true 的效果

TaskScheduler vs TaskExecutor

如果在两个 Configuration 类中存在 methodName 一样的 @Bean method, 会怎么样呢？

SpringBoot web 如何添加 ServerContext 初始化参数

Spring Framework 发布事件的顺序，及监听器的执行情况

集合 Bean 在 Spring 中是如何管理的

# 经验

* 在比 application.yml 优先级更高的属性源中设置 spring.profiles.active
* 不要在 application-{profile}.yml 中设置 spring.profiles.active