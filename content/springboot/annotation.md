---
title: "springboot-annotation"
date: 2019-08-15T13:19:18+08:00
draft: true
---



**@ConfigurationProperties**

> 根据properties注入属性生成bean，支持嵌套

常用属性

1. prefix 注入该前缀的属性
2. ignoreUnknownFields 是否忽略未知属性

参考链接:https://blog.csdn.net/lafengwnagzi/article/details/55050677

**@ConditionalOnExpression**

> 在指定条件下初始化bean

常用属性

1. value 为`true`时初始化

**@SpringBootTest**

配置文件属性的读取

可以在运行Spring引导测试的测试类上指定的注释。在常规Spring TestContext框架之上和之上提供以下特性:

当定义没有特定的@ContextConfiguration(loader=…)时，使用SpringBootContextLoader作为默认的ContextLoader。

当不使用嵌套@Configuration时，自动搜索@SpringBootConfiguration，并且没有指定显式的类。

允许使用properties属性定义自定义环境属性。

为不同的webEnvironment模式提供支持，包括启动一个完全运行的web服务器，监听一个已定义的或随机的端口。

注册一个TestRestTemplate和/或WebTestClient bean，用于在web测试中使用完全运行的web服务器。

**CommandLineRunner**

> 项目启动时会去运行实现这个接口的所有实现。也可以输入参数
>
> https://blog.csdn.net/catoop/article/details/50501710

