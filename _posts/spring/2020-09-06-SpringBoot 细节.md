---
layout:     post
title:      SpringBoot 细节
subtitle:   SpringBoot 细节
date:       2020-09-06
author:     Eagle Cool
header-img: img/post-bg-ios9-web.jpg
catalog: 	true
tags:       Spring
---

# ConditionEvaluator#shouldSkip()

CONFIGURATION_CLASS_FULL or CONFIGURATION_CLASS_LITE；
PARSE_CONFIGURATION 会先于 REGISTER_BEAN 被解析。

## 会跳过
* 从 @Conditional 中获取所有的 Condition
* 如果是 ConfigurationCondition；有一个 ConfigurationCondition 返回的 ConfigurationPhase 为 null 或者与参数 phase 相等，并且 condition 不匹配

# 获取自动配置类的主要逻辑

* ${spring.boot.enableautoconfiguration:true} 必须为 true
* 获取 SpringFactories’s EnableAutoConfiguration --> configurations
* 使用 LinkedHashSet 将 configurations 去重
* 获取需要排除的配置：EnableAutoConfiguration#exclude + EnableAutoConfiguration#excludeName + ${spring.autoconfigure.exclude}
* 校验需要排除的配置是否有效：即，类路径和 configurations 中是否存在需要排除的配置
* 从 configurations 中删除所有需要排除的配置
* 对 configurations 使用 SpringFactories’s AutoConfigurationImportFilter 判断是否应该跳过配置
    * OnClassCondition：
        * 使用 autoConfigurationClassName.ConditionalOnClass 从 META-INF/spring-autoconfigure-metadata.properties 中获取值（类名集合）
        * 获取到的类名必须在类路径中
    * OnWebApplicationCondition：
        * 使用 autoConfigurationClassName.ConditionalOnWebApplication 从 META-INF/spring-autoconfigure-metadata.properties 中获取值（ANY, SERVLET, REACTIVE）
            * ANY: 类路径中存在 org.springframework.web.context.support.GenericWebApplicationContext or org.springframework.web.reactive.HandlerResult
            * SERVLET: 类路径中存在 org.springframework.web.context.support.GenericWebApplicationContext
            * REACTIVE: 类路径中存在 org.springframework.web.reactive.HandlerResult
    * OnBeanCondition：
        * 使用 autoConfigurationClassName.ConditionalOnBean 从 META-INF/spring-autoconfigure-metadata.properties 中获取值（类名集合）
        * 获取到的类名必须出现在类路径中
        * 使用 autoConfigurationClassName.ConditionalOnSingleCandidate 从 META-INF/spring-autoconfigure-metadata.properties 中获取值（类名集合）
        * 获取到的类名必须出现在类路径中
* fireAutoConfigurationImportEvents
    * 获取 SpringFactories’s AutoConfigurationImportListener(ConditionEvaluationReportAutoConfigurationImportListener)
    * 发布 AutoConfigurationImportEvent 事件
    * 使用 ConditionEvaluationReport 记录匹配的 Configuration 和排除的 Configuration
* 将 configurations 排序
    * 按 字母顺序 排序
    * 按 @AutoConfigureOrder or META-INF/spring-autoconfigure-metadata.properties 中的 autoConfigurationClassName.AutoConfigureOrder 的顺序排序
    * 按 AutoConfigureAfter 排序
        * 根据 @AutoConfigureBefore @AutoConfigureAfter 收集 before and after 的关系
        * 从 META-INF/spring-autoconfigure-metadata.properties 中的 autoConfigurationClassName.AutoConfigureBefore autoConfigurationClassName.AutoConfigureAfter 收集 before and after 的关系
        * 合并上面收集的 before and after 关系
        * 根据 before 关系反推除 after 的关系
        * 按 after 关系排序
* 把 configurations 按照 @Import 处理

# Condition 条件

![](https://s1.ax1x.com/2020/09/08/wQICdJ.png)

## SpringBootCondition

* 通过模板方法 getMatchOutcome 获取 ConditionOutcome
* 打印 ConditionOutcome 日志
* 将 ConditionOutcome 记录到 ConditionEvaluationReport 中

##### OnClassCondition

* 指定的类必须在(@ConditionalOnClass)或(@ConditionalOnMissingClass)不在类路径上

##### OnWebApplicationCondition

* 当前应用必须是(@ConditionalOnWebApplication)或不是(@ConditionalOnNotWebApplication)指定的 ConditionalOnWebApplication$Type
    * SERVLET: 
        * 类路径中必须存在 org.springframework.web.context.support.GenericWebApplicationContext &&
        * BeanFactory 注册了 session scope ||
        * context.getEnvironment() instanceof ConfigurableWebEnvironment ||
        * context.getResourceLoader() instanceof WebApplicationContext
    * REACTIVE: 类路径中存在 org.springframework.web.reactive.HandlerResult

##### OnBeanCondition

* 指定的 Bean 是否存在与 BeanFactory 中
* ConfigurationPhase 为 REGISTER_BEAN

##### OnPropertyCondition

* 如果指定的 prefix + "." + name 属性不存在, 使用 matchIfMissing 判断
* 如果指定的 prefix + "." + name 属性存在, 使用 havingValue 判断

### AbstractSessionCondition

### ResourceCondition

#### HazelcastConfigResourceCondition

## ConfigurationCondition

### AbstractNestedCondition

#### NoneNestedConditions

所有的 Condition 都不匹配

#### AllNestedConditions

所有的 Condition 都匹配

#### AnyNestedCondition

任意一个 Condition 匹配

# ConfigurationClass 的处理过程

* 处理 MemberClasses
* 处理 @PropertySource
* 处理 @ComponentScan
* 处理 @Import
* 处理 @ImportResource
* 处理 @Bean 
* 处理 Interfaces‘s default methods
* 返回父类，递归处理


# bean 的创建过程

## 创建前 

* 转换 beanName
    * 去掉 "&" 前缀
    * 如果 beanName 是别名，替换为真名
* 从 DefaultSingletonBeanRegistry#singletonObjects 中获取对象，如果存在直接返回
* 如果 beanName 是 prototype，并且已经在创建中；抛出异常
* 如果不仅仅检查 type，
    * 将 bd.stale = true
    * 从 DefaultListableBeanFactory#mergedBeanDefinitionHolders 中删除 beanName
    * 将 beanName 添加到 AbstractBeanFactory#alreadyCreated 中；
* 使用 getMergedLocalBeanDefinition 获取 bd
    * 从 DefaultListableBeanFactory#beanDefinitionMap 中获取 bd
    * 从 AbstractBeanFactory#mergedBeanDefinitions 中获取 mbd
    * 如果 bd 是 RootBeanDefinition，克隆 bd 为 newBd；否则使用 bd new 一个 RootBeanDefinition 为 newBd
    * 如果 newBd 没有 Scope，设置为 singleton
    * 将 newBd 添加到 AbstractBeanFactory#mergedBeanDefinitions 中
    * 如果 newBd 的 targetType 与 mbd 的一样，将 mbd 的 targetType，isFactoryBean，resolvedTargetType，factoryMethodReturnType，factoryMethodToIntrospect 赋值给 newBd
    * 返回 newBd
* 如果 bd.isAbstract 抛出异常
* 如果 bd.dependsOn 不为空，先创建 bd.dependsOn
    * 注册依赖关系
    * 递归创建 dependsOn bean
* 根据 bd scope 的不同分别创建 singleton，prototype 和 其它

## 创建中

### 创建 singleton

* 如果 DefaultSingletonBeanRegistry#inCreationCheckExclusions 不包含 beanName && DefaultSingletonBeanRegistry#singletonsCurrentlyInCreation 已经包含 beanName，抛出异常
* 解析 bd 的 beanClass
    * 如果 AbstractBeanDefinition#beanClass 是 Class, 直接使用 AbstractBeanDefinition#beanClass
    * 否则 
* 准备 bd 的 method overrides
* 给 BeanPostProcessors 一个机会返回一个代理去取代目标 bean 实例
    * 如果 RootBeanDefinition#beforeInstantiationResolved != Boolean.FALSE && AbstractBeanDefinition#synthetic == false && AbstractBeanFactory#hasInstantiationAwareBeanPostProcessors == true
        * 推测目标类型 targetType，如果 bd 的 targetType 不为 null，直接使用
    * 遍历 AbstractBeanFactory#beanPostProcessors 中的 InstantiationAwareBeanPostProcessor
        * AnnotationAwareAspectJAutoProxyCreator：使用 AOP 代理包装每一个合格的 bean，在调用 bean 本身之前委托给指定的拦截器
    * 如果有一个 InstantiationAwareBeanPostProcessor 返回的结果不为 null，使用该结果代替目标 bean
    * 如果目标对象被成功代理，执行 BeanPostProcessor#postProcessAfterInitialization
* 使用 beanName 从 AbstractAutowireCapableBeanFactory#factoryBeanInstanceCache 中获取 instanceWrapper
* 如果 instanceWrapper == null，重新创建一个 instanceWrapper
    * 解析 bd 的 beanClass
    * 如果 AbstractBeanDefinition#instanceSupplier 不为 null，使用 instanceSupplier 获取 instanceWrapper
    * 如果 AbstractBeanDefinition#factoryMethodName 不为 null，使用 factoryMethodName 获取 instanceWrapper
        * 创建一个 ConstructorResolver 去使用 factoryMethod 实例化 instanceWrapper
            * 直接创建 BeanWrapperImpl
                * TypeConverterSupport#typeConverterDelegate
            * 使用 beanFactory 初始化 BeanWrapperImpl
                * PropertyEditorRegistrySupport#conversionService = AbstractBeanFactory#conversionService
                * baseEditor = new ResourceEditor(this.resourceLoader, this.propertyResolver)
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(Resource.class, baseEditor)
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(ContextResource.class, baseEditor)
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(InputStream.class, new InputStreamEditor(baseEditor))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(InputSource.class, new InputSourceEditor(baseEditor))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(File.class, new FileEditor(baseEditor))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(Path.class, new PathEditor(baseEditor))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(Reader.class, new ReaderEditor(baseEditor))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(URL.class, new URLEditor(baseEditor))
                * classLoader = this.resourceLoader.getClassLoader()
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(URI.class, new URIEditor(classLoader))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(Class.class, new ClassEditor(classLoader))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(Class[].class, new ClassArrayEditor(classLoader))
                * PropertyEditorRegistrySupport#overriddenDefaultEditors.put(Resource[].class, new ResourceArrayPropertyEditor(this.resourceLoader, this.propertyResolver))
            * 获取 bd 的 factoryBeanName，factoryBeanName 不能等于 beanName
            * 使用 factoryBeanName 从 beanFactory 中获取 factoryBean
            TODO
    * 从 BeanPostProcessors 中推测 Constructor
        * AutowiredAnnotationBeanPostProcessor
            * 如果 beanClass 中有 @Lookup 注解，将被 @Lookup 注解的方法构造为 LookupOverride，添加到 AbstractBeanDefinition#methodOverrides 中
            * 如果 beanClass 中有 被 @Autowired or @Value 注解的 Constructor，收集所有 被 @Autowired or @Value 注解的 Constructor 和默认的 Constructor 返回  
            * 如果 beanClass 只有一个 Constructor，将其返回
            * 如果 beanClass 有多个 Constructor，返回 空数组
    * 解析 bd 的 AutowireMode
        * 如果 AbstractBeanDefinition#autowireMode == AUTOWIRE_AUTODETECT
            * beanClass 有默认 Constructor，返回 AUTOWIRE_BY_TYPE
            * 否则返回 AUTOWIRE_CONSTRUCTOR
        * 否则，返回 AbstractBeanDefinition#autowireMode
    * 如果推测出 Constructor，或者 AutowireMode == AUTOWIRE_CONSTRUCTOR，或者 AbstractBeanDefinition#constructorArgumentValues 不为空；使用 autowireConstructor 创建 instanceWrapper
        * 创建一个 ConstructorResolver 去使用 constructor 实例化 instanceWrapper
            * 直接创建 BeanWrapperImpl
            * 使用 beanFactory 初始化 BeanWrapperImpl
            * 如果 beanClass 只有一个 ConstructorA 并且 AbstractBeanDefinition#constructorArgumentValues 为空




