---
title: "dailyrecord"
date: 2019-08-15T13:19:18+08:00
draft: true

---

#### web.xml中 `classpath` 和 `classpath*` 的区别

classpath 是指WEB-INF下的classes目录，所有src目录下的java，xml，properties文件编译后都会放在classes目录下
classpath：只会到你的class路径中查找文件
classpath*：会将所有可用的class路径全部遍历加载，当项目中有多个classpath路径，并同时加载多个classpath路径下（此种情况多数不会遇到）的文件，*就发挥了作用，如果不加*，则表示仅仅加载第一个classpath路径。

> 类似于下图结构，config目录与src同级，此时就需要通过`classpath*`进行加载，不然无法加载到配置文件
![classpath结构](http://op3iwn4h5.bkt.clouddn.com/18-4-20/38534219.jpg)


#### @Slf4j注解
`@Slf4j`是`lombok`的便利开发的注解。在类上使用该注解后可以不用新建`Logger`实例
`@Slf4j(topic = "com.souche.erp.dq")`定义logger的类别，默认是使用被注解类的类别

> 5.7
#### mybatis
尽量单表查询,对于mybatis输出的集合,利用`Collections.isNotEmpty()`判断是否为空
#### fastjson序列化输出null值
这是fastjson默认的输出策略，如果值为null则不输出，可以通过`SerializerFeature`来进行修改，主要常用`WriteMapNullValue`,即输出值为null的值


