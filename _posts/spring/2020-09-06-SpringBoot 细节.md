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

