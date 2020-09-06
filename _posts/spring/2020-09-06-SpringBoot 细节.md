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


