---
title: "interview"
date: 2019-08-15T13:19:18+08:00
draft: true
---

金三银四的时候，面初级经验少真的是有些尴尬。查缺补漏，做到最好。
<!--more-->

### 奇码信息
数据库查询多练习
多对多，一对多关系
约瑟夫问题，N个人围成圈，数到3的人跳出去，问最后留下谁
项目架构？纵向，横向
写代码时有哪些禁忌
各个子系统如何垮域
Http协议介绍
get post区别
Nginx负载均衡配置

### 居然之家
springboot如何搭建web项目
添加web模块依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

restful如何解释
retful是指基于rest的服务接口风格。通过URL定位资源，HTTP的method定义操作类型。restful最大的有点在于统一接口，多个client都可以使用同一套接口访问
springboot使用@springbootapplication
该注解可以代替**@Configuration, @EnableAutoConfiguration,  @ComponentScan** 
springboot中SpringApplication类的作用
程序的入口类，可以使用该类的run方法在main方法下进行调用，启动程序

spring web的实现原理
servlet的实现原理 [看这个](https://www.jianshu.com/p/c63f4ee66117)
spring di的实现原理

描述单点登录实现
编写去除数据库中重复数据只保留一条
```sql
delete from table where id not in (select min(id) from table group by name having count(name)>1) and  id in (select id group by name having count(name)>1)
```
递归，迭代区别


### 大搜车
**springmvc的工作机制**
![工作机制](http://img.my.csdn.net/uploads/201211/16/1353059506_5137.jpg)
**springmvc的常用注解**
[戳这里](https://www.cnblogs.com/leskang/p/5445698.html)
**@ResponseBody作用**
 该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。主要用于返回json，xml等特殊格式时使用，不加该注解则返回html文档
**@Resource和@Autowired有什么区别**
相同点：两个注解都可以实现在属性上，set方法上注入
不同点：
@Autowired是spring提供的注解，只支持byType类型注入，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。eg:
```
public class TestServiceImpl {
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao; 
}
```
@Resource是J2EE提供的注解，默认为ByName类型注入
**spring的ioc/di机制**
**spring管理的对象是单例还是多例，如何修改**
默认单例(singleton)，可通过**scope**属性更改，创建多例对象后spring比不会维护其生命周期
**aop机制**
AOP（Aspect Orient Programming），我们一般称为面向方面（切面）编程，作为面向对象的一种补充，用于处理系统中分布于各个模块的横切关注点，比如事务管理、日志、缓存等等。AOP实现的关键在于AOP框架自动创建的AOP代理，AOP代理主要分为静态代理和动态代理，静态代理的代表为AspectJ；而动态代理则以Spring AOP为代表。本文会分别对AspectJ和Spring AOP的实现进行分析和介绍。
[戳这里](http://www.importnew.com/24305.html)
**beanfactory和applicationcontext之间的关系**
org.springframework.beans及org.springframework.context包是Spring IoC容器的基础。BeanFactory提供的高级配置机制，使得管理任何性质的对象成为可能。ApplicationContext是BeanFactory的扩展，功能得到了进一步增强，比如更易与Spring AOP集成、消息资源处理(国际化处理)、事件传递及各种不同应用层的context实现(如针对web应用的WebApplicationContext)。 简而言之，BeanFactory提供了配制框架及基本功能，而ApplicationContext则增加了更多支持企业核心内容的功能。ApplicationContext完全由BeanFactory扩展而来，因而BeanFactory所具备的能力和行为也适用于ApplicationContext。
**spring创建对象的流程**
1.Spring容器启动时，首先会加载spring的核心配置文件。
2.Spring器逐行进行解析xml配置文件
3.当解析bean标签时开始创建对象，根据class属性中的数据通过反射机制创建对象。
4.将创建好的对象存入Spring所维护的Map<key,value>中 key就是bean中的ID.
value就是生成的对象。

**Spring的传播机制，主要是指事务处理时的配置的事务级别有哪些，默认的级别主要有什么作用，给了一个事务嵌套的问题进行深入，a事务嵌套b事务时，会出现的各种抛异常的问题，是否会触发回滚**
[戳这里--深入理解spring事务](https://blog.csdn.net/yuanlaishini2010/article/details/45792069)  


**mybatis中#{}和${}区别，还给了一个动态sql判断预编译的结果**
[戳这里](https://segmentfault.com/a/1190000004617028)
**mybatis动态编写xml时如何实现传进来的值不为空时才编译进去，要用到什么标签**
[mybatis动态更新](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html)
**事务acid属性**
A原子性  
C一致性 
I隔离性  
D持久性  
[戳这里](http://www.cnblogs.com/zhaoyl/p/4288752.html)
**写了个关联查询，两张表**
**如何打日志**
[戳这里](http://www.importnew.com/28024.html)
