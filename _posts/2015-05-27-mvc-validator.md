---
title: 记一次在spring-mvc中踩坑的经历
tags: spring mvc java
categories: java
published: true
---

今天打算在项目中加入数据验证功能，具体可参考 [数据验证][validator]

过程无须赘述

配置完成后，奇怪的现象发生了，`BindingResult.hasErrors()`永远返回 false
![hasErrors][haserrors]

这是什么情况? validator 未指定？我赶紧检查配置：
![notNull][notnull]

![driven][driven]
配置没有问题，日志能够正常打印，且没有错误，排除类加载失败的可能。

上 google 搜索一下！
![google][google]
翻看了第一页的回复，大多来自 stackoverflow，无非是让你引入包, 加配置, 我项目里配置符合这些回复的要求, 所以需要另想办法

自己动手吧!我加入如下代码，以验证 validator 是否被正确加载
![initBinder][initbinder]
validator 果然为 null, 且如果拿到 validator 对象并 set 进去，就可以解决`BindingResult.hasErrors()`永远返回 false 的问题,
然而，这种处理方式需要在每个 Controller 中写一遍 initBinder, 这种方式并不完美。

是什么原因造成 validator 丢失呢?
我们知道`annotation-driven`会在 spring 中自动注册多个 bean, 详情请移步[自动注册][自动注册]
会不会是项目中存在某些配置覆盖了其中的 bean，造成了 validator 丢失?
仔细扫描一下配置文件, 发现果然如此, 项目中为了更换 json 转换器, 自定义了一个 bean
![adapter][adapter]

到了这个地步基本上找到原因了，解决方式很简单，将 validator 绑定起来就可以了
![validator-config][validator-config]

[validator]: http://jinnianshilongnian.iteye.com/blog/1733708
[自动注册]: https://my.oschina.net/HeliosFly/blog/205343

[hasErrors]:{{"/mvc-validator/hasErrors.jpg" | prepend: site.imgrepo }}
[validator-config]:{{"/mvc-validator/validator-config.jpg" | prepend: site.imgrepo }}
[google]:{{"/mvc-validator/google.jpg" | prepend: site.imgrepo }}
[initBinder]:{{"/mvc-validator/initBinder.jpg" | prepend: site.imgrepo }}
[adapter]:{{"/mvc-validator/adapter.jpg" | prepend: site.imgrepo }}
[driven]:{{"/mvc-validator/driven.jpg" | prepend: site.imgrepo }}
[notNull]:{{"/mvc-validator/notNull.jpg" | prepend: site.imgrepo }}
