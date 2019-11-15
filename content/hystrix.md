---
title: "hystrix"
date: 2019-08-15T13:19:18+08:00
draft: true
---


## 运行流程图

![来自hystrix的github站点](https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/hystrix-command-flow-chart.png)

1. 构建一个请求实体

   - `HystrixCommand` 有简单的 response
   - `HystrixObservableCommand` 可观察的 response

2. 执行`Command`

   有四种方式来执行命令,前两种只适用于`HystrixCommand`

   1. `execute()` 同步返回响应结果或者报错
   2. `queue()` 返回`Future`装饰的结果
   3. `observe()` 订阅`Observable`,直接执行并返回其副本
   4. `toObservable()` 返回一个`Observable`，订阅时就执行并发送结果

   以上四种方式最后都会执行到`toObservable`方法，因为· `HystrixCommand`实现的是`Observable`接口

3. 是否有 Response 缓存？

4. 断路器是否打开？

## 隔离机制

> bulkhead pattern (隔板/舱壁 模式)

## 构造参数

### Command Name

### Command Group

通过组来展示

### Command Thread-Pool

线程池命令，通过它来操作监控，各类指标，缓存的用途

用这个的目的可能是想隔离开线程之间的问题

### Request Cache

缓存请求，如果要是用的话有两点要注意

1. 需要重写`getCacheKey()`
2. 需要初始化`HystrixRequestContext`,通常情况下 context 会通过`ServletFilter`完成其生命周期(初始化、销毁),或者是其他的生命周期 hook

### Request Collapsing

支持多个请求批量执行

### Request Context Setup

要使用请求作用域的特性则需要管理`HystrixRequestContext`的生命周期或者实现`HystrixConcurrencyStrategy`

> 初始化后要记得 shut down

标准的 Java 应用,可以使用 Servlet Filter 来做这件事

```java
public class HystrixRequestContextServletFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
     throws IOException, ServletException {
        HystrixRequestContext context = HystrixRequestContext.initializeContext();
        try {
            chain.doFilter(request, response);
        } finally {
            context.shutdown();
        }
    }
}
```

### Common Patterns

经常使用的几种模式

1. Fail Fast 快速失败

   直接执行,没有回滚逻辑,有错就抛

2. Fail Silent 失败就返回默认返回

   一般就实现`getFallback`返回 null,emptyList(),emptyMap()

3. Fallback: Static 感觉其实和 Fail Silent 一致

4. Fallback: Stubbed

   自行组装 fallback

5. Fallback: Cache via Network

   通过缓存获取 fallback

## 如何配置并开启熔断

> 这是网飞自己的实践经验总结,规模在 100+command types,40+tread pool,每天 100 亿+thread-isolated,2000 亿+semaphore-isolated 的命令执行

1. 保留默认的 1000ms 超时,除非知道需要更多时间
2. 保留默认的线程池容量(10),除非知道需要更多的线程
3. 发布到[金丝雀(这个不太清楚)](https://jubel.cn/posts/automated-canary-analysis-tool-kayenta),没问题就继续
4. 到生产环境跑 24 小时
5. 依靠报警和监控来定位问题(如果有的话)
6. 24 小时候,通过响应时间的比例和流量来计算断路器合理的最小配置
7. 在生产环境热更新配置并通过监控平台实时监控
8. 只有当系统的性能发生变化或者发出警报等引起你注意时,才能再次修改配置

官方也给出了一个配置的例子

![如图](https://github.com/Netflix/Hystrix/wiki/images/thread-configuration-640.png)

解释下图上的参数

1. 线程池大小怎么计算出来的?

假设每秒请求为 30 次,默认线程池大小为 10,一次正常返回时长假设为 200ms,再加上一个 buffer 作为缓冲得到的就是

$$30 * 0.2 + 4 = 10$$

2. 超时时间为什么是 300ms?

一次正常请求可能在 200ms,在有点超时的情况下,就再多加 50ms,如果超时重试了,取一个超时重试的平均时延假设 40ms,差不多就是 300ms 了.

如果再设置大一点呢?比如 350ms,如果一次请求耗费时长超多了 350ms,那在 1s 中每个线程只能处理不到两个请求,10 个线程就处理不到 30 个请求,这 1s 中的还未处理的请求就会被拒绝掉,下一秒进来的请求由于上一秒的请求导致线程还在阻塞也会被拒绝掉.

大多数的断路器,需要将超时的时间设置在接近 99.5%的正常请求的时延位置,这样可以保证切断异常的请求,不对系统造成影响

## 解读 Dashboard

hystrix 有自己的 dashboard,根据 commandKey 来展示每一个 command 对应的 metrics 信息

![官网上的dashboard](https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/dashboard-annoted-circuit-640.png)

## 啥是响应式编程?

> 摘自维基百科

在计算机中，**响应式编程**或**反应式编程**（英语：Reactive programming）是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。

# 前言

hystrix 是我司目前使用的熔断限流的一种方案,虽然 Netflix 已不再维护这个项目,但还是足够稳定的.

开启断路器有以下几种情况:

1. 强制开启断路器

2. 一个周期内请求数量超过设置的断路器请求阈值(`circuitBreakerRequestVolumeThreshold`)

3. 错误请求占比超过设置的断路器错误请求占比(`circuitBreakerErrorThresholdPercentage`)

   计算公式: $failure / (failure + success)$

针对于第三种情况,需要计算的指标是根据请求的情况实时上报得来的,metrics 上报的特性是利用类似`滑动窗口`的思想来实现的.下面就来研究一下 hystrix 是如何实现的.

我们主要关注`AbstractCommand`中的`toObservable`方法,从官方 wiki 的流程图中我们也能知道,最后的逻辑都会走到这里,也是在这里生成了我们执行整个核心流程的`Observable`对象..也可以看到图中`report metrics`的虚线,其实就是我们主要关注的逻辑.

![来自hystrix的github站点](https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/hystrix-command-flow-chart.png)

# Hystrix 扫盲贴

## **总览**

我司项目里在远程调用的时候会利用[Netflix Hystrix](https://github.com/Netflix/Hystrix)进行限流熔断的保护处理,自己在了解 springcloud 的时候就已经知道大名鼎鼎的 hystrix 了,由网飞(Netflix)开源,是现在保障微服务可用性必不可少的一环.但一直没有使用和了解过,所以这次就从小白开始入门了.

## **什么是 Hystrix**?

> Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.

Hystrix 是一个延迟和容错库，旨在隔离对远程系统、服务和第三方库的访问点，停止级联故障，并在错误不可避免的复杂分布式系统中能够弹性恢复。

### **「友商」**

> 目前 hystrix 已经停止维护了,所以要是组件出了点 bug 什么的,就得自己搞…不过经过了这么久的维护以后,bug 应该也找的差不多了吧?

现在开源的生态里还有其他几款类似的容错的组件,后面也打算了解一下.

1. [Sentinel: 分布式系统的流量防卫兵](https://github.com/alibaba/Sentinel) ==> 阿里出品,特征摘自官方 wiki
1. **丰富的应用场景**:有阿里十年双十一流量洗礼的背书
2. **完备的实时监控**:大板能力
3. **广泛的开源生态**:深入继承了各类框架
4. **完善的 SPI 扩展点**:二次开发便利
2. [resilience4j](https://github.com/resilience4j/resilience4j)==>官方介绍为:一款灵感来自[Netflix Hystrix](https://github.com/Netflix/Hystrix)的轻量级容错框架,基于`Java8`和`函数式编程`.

### 组件比较

> 摘自 [sentinel 官方 wiki](<[https://github.com/alibaba/Sentinel/wiki/Guideline:-%E4%BB%8E-Hystrix-%E8%BF%81%E7%A7%BB%E5%88%B0-Sentinel](https://github.com/alibaba/Sentinel/wiki/Guideline:-从-Hystrix-迁移到-Sentinel)>)

|                | Sentinel                                                   | Hystrix                 | resilience4j                     |
| -------------- | ---------------------------------------------------------- | ----------------------- | -------------------------------- |
| 隔离策略       | 信号量隔离（并发线程数限流）                               | 线程池隔离/信号量隔离   | 信号量隔离                       |
| 熔断降级策略   | 基于响应时间、异常比率、异常数                             | 基于异常比率            | 基于异常比率、响应时间           |
| 实时统计实现   | 滑动窗口（LeapArray）                                      | 滑动窗口（基于 RxJava） | Ring Bit Buffer                  |
| 动态规则配置   | 支持多种数据源                                             | 支持多种数据源          | 有限支持                         |
| 扩展性         | 多个扩展点                                                 | 插件的形式              | 接口的形式                       |
| 基于注解的支持 | 支持                                                       | 支持                    | 支持                             |
| 限流           | 基于 QPS，支持基于调用关系的限流                           | 有限的支持              | Rate Limiter                     |
| 流量整形       | 支持预热模式、匀速器模式、预热排队模式                     | 不支持                  | 简单的 Rate Limiter 模式         |
| 系统自适应保护 | 支持                                                       | 不支持                  | 不支持                           |
| 控制台         | 提供开箱即用的控制台，可配置规则、查看秒级监控、机器发现等 | 简单的监控查看          | 不提供控制台，可对接其它监控系统 |

---

## Hello Hystrix

> 这里引用官方的 demo

```java
public class CommandHelloWorld extends HystrixCommand<String> {

  private final String name;

  public CommandHelloWorld(String name) {
    super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
    this.name = name;
  }

  @Override
  protected String run() {
    return "Hello " + name + "!";
  }

  public static void main(String[] args) {
    // 1.同步返回
    String s = new CommandHelloWorld("Bob").execute();
    // 2.异步返回结果
    Future<String> s = new CommandHelloWorld("Bob").queue();
    // 3.生成Observable对象,subscribe后才执行
    Observable<String> s = new CommandHelloWorld("Bob").observe();
  }
}
```

## Hystrix 运行流程

> 还是引用自官方文档

![来自hystrix的github站点](https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/hystrix-command-flow-chart.png)

流程图大致介绍:

1. 创建一个 HystrixCommand/HystrixObservableCommand

两种类型的 command 没有太大的区别,都是需要重写 run/construct 方法来完成自己的业务逻辑

2. 执行方法,最后都会调用`toObservable`方法,该方法是真正生成`Observable`的逻辑实现

3. 当有请求进来,会先判断是否有缓存结果,有则返回,无则继续下一步

4. 判断断路器(`circuit-breaker`)是否开启,若已开启则执行短路,执行第 8 步

5. 断路器未开启时,判断是否会被线程池或信号量拒绝(即请求饱和的情况),不拒绝则进行第 6 步,拒绝则进行第 8 步,同时执行第七步上报执行结果

6. 程序执行到这里即为执行我们想要处理的业务逻辑,如果执行异常或者超时时会调用第 8 步,否则执行第 9 步,并上报执行结果

7. 上报结果主要为执行是否成功,是否短路,拒绝等情况,形成综合指标作为断路器判断是否开启的依据

8. 执行失败回滚的逻辑,如果失败回滚成功,返回回滚结果,否则做其他处理

9. 正常执行结果返回

在整个执行过程中,需要业务放处理的只有两处

1. 第 6 步重写 run/construct 方法
2. 第 8 步重写 getFallback/resumeWithFallback 方法完成异常逻辑处理(如果需要的话)

## 核心概念

### Command 命令

**Command**是Hystrix的入口,对用户来说,我们只需要创建对应的command,将需要保护的接口包装起来就可以.可以无需关注再之后的逻辑.与Spring深度集成后还可以通过注解的方式,就更加对开发友好了.

### Circuit Breaker 断路器

断路器,是从电气领域引申过来的概念,具有过载、短路和欠电压保护功能，有保护线路和电源的能力.在软件领域则引申为请求比例超过一定阈值后则对后面的请求直接fail保障系统的稳定,还会尝试检测服务是否恢复,若恢复则会关闭断路器.

hystrix中断路器的执行流程官方有很直观的流程图:

![circuit-breaker](https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/circuit-breaker-1280.png)

可以看出流程图中断路器开闭的计算方式是 $failure / (failure + success)$ > `threshold`(通过配置`circuitBreakerErrorThresholdPercentage`参数设置,默认为`50`),如果大于则开启断路器,开启断路器后由于需要尝试判断系统是否恢复,在短路一段时间后,会放行请求去执行,执行成功则会关闭断路器并且重置计数

那么`failure`和`success`的数据(下方的两个长方形的图例)如何统计来呢,hystrix利用rxjava巧妙的实现了滑动窗口来实时的统计失败成功的请求数量.

在我们执行成功或者失败时,都会上报进行统计



### Isolation 隔离策略

hystrix利用[舱壁模式](https://docs.microsoft.com/en-us/azure/architecture/patterns/bulkhead)的思想来对访问的资源进行隔离,每个资源是独立的依赖,单个资源的异常不应该影响到其他.

![isolation](https://github.com/Netflix/Hystrix/wiki/images/soa-5-isolation-focused-640.png) [hystrix.md](hystrix.md) 

Hystrix提供两种隔离方式,**ThreadPool** 和 **Semaphore**,如下图所示,两种隔离模式的请求.

`DependencyA`,`DependencyI` 为线程池隔离模式,`DependencyX`为信号量隔离模式

![Isolation-strategy](https://raw.githubusercontent.com/wiki/Netflix/Hystrix/images/isolation-options-1280.png)

#### ThreadPool Isolation

当用户请求服务A和服务I的时候，容器(tomcat/jetty)的线程会将请求的任务交给服务A和服务I的内部线程池里面的线程来执行，tomcat的线程就可以去干别的事情去了，当服务A和服务I自己线程池里面的线程执行完任务之后，就会将调用的结果返回给tomcat的线程，从而实现资源的隔离，当有大量并发的时候，服务内部的线程池的数量就决定了整个服务的并发度，例如服务A的线程池大小为10个，当同时有12请求时，只会允许10个任务在执行，其他的任务被放在线程池队列中，或者是直接走降级服务，此时，如果服务A挂了，就不会造成大量的tomcat线程被服务A拖死，服务I依然能够提供服务。整个系统不会受太大的影响。

#### Semaphore Isolation

信号量的资源隔离只是起到一个开关的作用，例如，服务X的信号量大小为10，那么同时只允许10个tomcat的线程(此处是tomcat的线程，而不是服务X的独立线程池里面的线程)来访问服务X，其他的请求就会被拒绝，从而达到限流保护的作用。信号量的大小可以 **动态配置** .

### 创建 command

创建一个 command 即为上述的第一步,在`hystrix`中,很多逻辑都是在构造函数中完成的.由于观察者模式的特性,当 Observable 初始化完成后,其实就开始源源不断的产生影响了,所以构造函数需要好好研究下.

我们以官方 demo 作为切入点,先观察其类图

![WX20190724-155516.png](https://i.loli.net/2019/07/24/5d380efdeaac457770.png)

`AbstractCommand`实现了上述的大部分接口,完成了 Command 整体的逻辑,可以说是`Command`命令的核心类.

HystrixCommand 以及其他的 Command 是实现了其中带有各自特征的接口.处理不同的逻辑.

查看`CommandHelloWorld` 和`HystrixCommand`构造方法如下.可以看到其实都是调用上层的逻辑.所以可以直接在 AbstractCommand 查看

```java
public CommandHelloWorld(String name) {
super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
this.name = name;
}
protected HystrixCommand(HystrixCommandGroupKey group) {
super(group, null, null, null, null, null, null, null, null, null, null, null);
}
```

`AbstractCommand`初始化逻辑

```java
protected AbstractCommand(HystrixCommandGroupKey group, HystrixCommandKey key, HystrixThreadPoolKey threadPoolKey, HystrixCircuitBreaker circuitBreaker, HystrixThreadPool threadPool,
    HystrixCommandProperties.Setter commandPropertiesDefaults, HystrixThreadPoolProperties.Setter threadPoolPropertiesDefaults,
    HystrixCommandMetrics metrics, TryableSemaphore fallbackSemaphore, TryableSemaphore executionSemaphore,
    HystrixPropertiesStrategy propertiesStrategy, HystrixCommandExecutionHook executionHook) {
  // 初始化诸多属性,所有init方法逻辑都是一致的

  // 初始化groupKey,用以分开不同的组,默认情况下会作为TheadPoolKey
  this.commandGroup = initGroupKey(group);
  // 初始化commandKey,key为空则默认以className作为key
  this.commandKey = initCommandKey(key, getClass());
  // 初始化命令参数,是否允许请求缓存,metric上报周期等在这里进行设置
  this.properties = initCommandProperties(this.commandKey, propertiesStrategy, commandPropertiesDefaults);
  // 初始化线程池key
  this.threadPoolKey = initThreadPoolKey(threadPoolKey, this.commandGroup, this.properties.executionIsolationThreadPoolKeyOverride().get());
  // 初始化metrics上报
  this.metrics = initMetrics(metrics, this.commandGroup, this.threadPoolKey, this.commandKey, this.properties);
  // 初始化断路器
  this.circuitBreaker = initCircuitBreaker(this.properties.circuitBreakerEnabled().get(), circuitBreaker, this.commandGroup, this.commandKey, this.properties, this.metrics);
  // 初始化线程池
  this.threadPool = initThreadPool(threadPool, this.threadPoolKey, threadPoolPropertiesDefaults);

  //Strategies from plugins
  // 有插件时对插件进行初始化,这里的思路就是通过类加载机制动态加载,每个属性都有默认的实现
  this.eventNotifier = HystrixPlugins.getInstance().getEventNotifier();
  this.concurrencyStrategy = HystrixPlugins.getInstance().getConcurrencyStrategy();
  HystrixMetricsPublisherFactory.createOrRetrievePublisherForCommand(this.commandKey, this.commandGroup, this.metrics, this.circuitBreaker, this.properties);
  this.executionHook = initExecutionHook(executionHook);

  this.requestCache = HystrixRequestCache.getInstance(this.commandKey, this.concurrencyStrategy);
  this.currentRequestLog = initRequestLog(this.properties.requestLogEnabled().get(), this.concurrencyStrategy);

  /* fallback semaphore override if applicable */
  this.fallbackSemaphoreOverride = fallbackSemaphore;

  /* execution semaphore override if applicable */
  this.executionSemaphoreOverride = executionSemaphore;
}
```

这里我们选`initThreadPool`来看下是如何处理的

```java
private static HystrixThreadPool initThreadPool(HystrixThreadPool fromConstructor, HystrixThreadPoolKey threadPoolKey, HystrixThreadPoolProperties.Setter threadPoolPropertiesDefaults) {
    // 简单判空有则返回,没有则初始化
    if (fromConstructor == null) {
	// get the default implementation of HystrixThreadPool
	return HystrixThreadPool.Factory.getInstance(threadPoolKey, threadPoolPropertiesDefaults);
    } else {
	return fromConstructor;
    }
}

// HystrixThreadPool.Factory
// hystrix的属性类都有专门的静态工厂(Factory,Setter)来进行属性的设置,获取实例等,可读性更高
/* package */static class Factory {
  //这里没有使用HystrixThreadPoolKey实例来作为key,而是使用HystrixThreadPoolKey.name()是因为HystrixThreadPool只是个接口,无法保证得到的实例是否正确实现了hashcode/equals方法,并且也不希望为每个相同name的实例创建不同的线程池
  /* package */final static ConcurrentHashMap<String, HystrixThreadPool> threadPools = new ConcurrentHashMap<String, HystrixThreadPool>();
  /* package */static HystrixThreadPool getInstance(HystrixThreadPoolKey threadPoolKey, HystrixThreadPoolProperties.Setter propertiesBuilder) {
      // get the key to use instead of using the object itself so that if people forget to implement equals/hashcode things will still work
      String key = threadPoolKey.name();

      // this should find it for all but the first time
      // 先查看缓存是否有
      HystrixThreadPool previouslyCached = threadPools.get(key);
      if (previouslyCached != null) {
	  return previouslyCached;
      }

      // if we get here this is the first time so we need to initialize
      synchronized (HystrixThreadPool.class) {
	  if (!threadPools.containsKey(key)) {
	      // 进行初始化,这里是默认的实现
	      threadPools.put(key, new HystrixThreadPoolDefault(threadPoolKey, propertiesBuilder));
	  }
      }
      return threadPools.get(key);
  }
}

public HystrixThreadPoolDefault(HystrixThreadPoolKey threadPoolKey, HystrixThreadPoolProperties.Setter propertiesDefaults) {
  // 获取线程池相关配置
  this.properties = HystrixPropertiesFactory.getThreadPoolProperties(threadPoolKey, propertiesDefaults);
  // 通过strategy获取线程池
  HystrixConcurrencyStrategy concurrencyStrategy = HystrixPlugins.getInstance().getConcurrencyStrategy();
  this.queueSize = properties.maxQueueSize().get();
  this.queue = concurrencyStrategy.getBlockingQueue(queueSize);
  // 是否设置了自定义的最大queueSize,超过则拒绝
  if (properties.getAllowMaximumSizeToDivergeFromCoreSize()) {
      this.metrics = HystrixThreadPoolMetrics.getInstance(threadPoolKey,
	      concurrencyStrategy.getThreadPool(threadPoolKey, properties.coreSize(), properties.maximumSize(), properties.keepAliveTimeMinutes(), TimeUnit.MINUTES, queue),
	      properties);
      this.threadPool = this.metrics.getThreadPool();
  } else {
      // 创建线程池的metrics记录,也是在此利用concurrencyStrategy创建了线程池
      this.metrics = HystrixThreadPoolMetrics.getInstance(threadPoolKey,
	      concurrencyStrategy.getThreadPool(threadPoolKey, properties.coreSize(), properties.coreSize(), properties.keepAliveTimeMinutes(), TimeUnit.MINUTES, queue),
	      properties);
      this.threadPool = this.metrics.getThreadPool();
  }

  /* strategy: HystrixMetricsPublisherThreadPool */
  // 创建一个默认的metricsThreadPool,默认是空实现
  HystrixMetricsPublisherFactory.createOrRetrievePublisherForThreadPool(threadPoolKey, this.metrics, this.properties);
}

```

#### toObservable

创建好一个 Command,根据上面官方的流程图,我们可以看到最后都会调用`toObservable`方法来生成一个事件源供需要的`Observer`订阅.
`toObservable`在`AbstractCommand`类中,方法有很清晰的结构,我们可以分为几块来看.

1. 处理 Observable 在不同状态下对应的 Action

```java
    final Action0 terminateCommandCleanup = new Action0() {

	@Override
	public void call() {
	    if (_cmd.commandState.compareAndSet(CommandState.OBSERVABLE_CHAIN_CREATED, CommandState.TERMINAL)) {
		handleCommandEnd(false); //user code never ran
	    } else if (_cmd.commandState.compareAndSet(CommandState.USER_CODE_EXECUTED, CommandState.TERMINAL)) {
		handleCommandEnd(true); //user code did run
	    }
	}
    };

    //mark the command as CANCELLED and store the latency (in addition to standard cleanup)
    final Action0 unsubscribeCommandCleanup = new Action0() {
	@Override
	public void call() {
	    if (_cmd.commandState.compareAndSet(CommandState.OBSERVABLE_CHAIN_CREATED, CommandState.UNSUBSCRIBED)) {
		if (!_cmd.executionResult.containsTerminalEvent()) {
		    _cmd.eventNotifier.markEvent(HystrixEventType.CANCELLED, _cmd.commandKey);
		    _cmd.executionResultAtTimeOfCancellation = _cmd.executionResult
			    .addEvent((int) (System.currentTimeMillis() - _cmd.commandStartTimestamp), HystrixEventType.CANCELLED);
		}
		handleCommandEnd(false); //user code never ran
	    } else if (_cmd.commandState.compareAndSet(CommandState.USER_CODE_EXECUTED, CommandState.UNSUBSCRIBED)) {
		if (!_cmd.executionResult.containsTerminalEvent()) {
		    _cmd.eventNotifier.markEvent(HystrixEventType.CANCELLED, _cmd.commandKey);
		    _cmd.executionResultAtTimeOfCancellation = _cmd.executionResult
			    .addEvent((int) (System.currentTimeMillis() - _cmd.commandStartTimestamp), HystrixEventType.CANCELLED);
		}
		handleCommandEnd(true); //user code did run
	    }
	}
    };

    final Action0 fireOnCompletedHook = new Action0() {
	@Override
	public void call() {
	    try {
		executionHook.onSuccess(_cmd);
	    } catch (Throwable hookEx) {
		logger.warn("Error calling HystrixCommandExecutionHook.onSuccess", hookEx);
	    }
	}
    };
```

### metrics 统计

之前提到过hystrix 断路器的工作原理, 是通过比较失败请求占比来实现.这些失败或成功的数据是利用滑动窗口来实时计算汇总的.接下来就来看看Hystrix Metrics 是如何实现的



#### metrics.rollingStats.timeInMilliseconds

一个滑动窗口的时间

这个配置只有在初始化时有效.之后若再设置是不会生效的

一个窗口会分割成多个 `bucket`,利用这些buket进行滑动

默认 `10000` ms

#### metrics.rollingStats.numBuckets

这个参数决定了一个滑动窗口需要分割成多少个bucket,必须保证能被 `metrics.rollingStats.timeInMilliseconds` 整除,即满足  $metrics.rollingStats.timeInMilliseconds \  \%  \ metrics.rollingStats.numBuckets == 0$ 

默认 `10`

#### metrics初始化









#### 滑动窗口计算











## RxJava 知识储备

想要了解 hystrix 是如何实现容错的我们就需要看看 hystrix 的源码,但是刚看我就蒙蔽了,hystrix 是完全基于`rxjava`实现的,在实现实时的统计时,也是利用了`rxjava`的语法糖很巧妙的实现了滑动窗口,所以在了解运行机制之前我们需要先做一些知识储备.

### 什么是 rxjava?

> 需要注意的是 rxjava 已经出到 3.x 的版本了,但 hystrix 中还是使用的 1.x 版本

#### 官网介绍

1. [RxJava](https://github.com/ReactiveX/RxJava)是响应式插件([[Reactive Extensions](http://reactivex.io/)])思想的 Java 实现.

2. Reactive Extentsions: 一个利用可观察序列来编写异步和事件驱动程序的库.

> It extends [the observer pattern](http://en.wikipedia.org/wiki/Observer_pattern) to support sequences of data and/or events and adds operators that allow you to compose sequences together declaratively while abstracting away concerns about things like low-level threading, synchronization, thread-safety, concurrent data structures, and non-blocking I/O.

它扩展了观察者模式,支持数据(data)或事件(events)的序列,并且可以声明式的添加用于组合序列的操作符(operators),同时抽象出了对低级线程,同步,线程安全,并发数据结构和非阻塞式 io 等问题的关注.

#### Concept

1. Observable — 可观察者(事件源)

响应式编程通过事件驱动,可观察者作为事件源不断的发送事件,交由观察者(订阅者)做处理

2. Observer — 观察者(订阅者)

观察者通过订阅(`subscribe`)与事件源建立连接,一旦 Observable 有事件发出就会触发订阅的逻辑.标准的 Observer 有三个触发逻辑需要实现.

- onNext

 当 Observable 发出事件时就会调用这个方法完成业务逻辑

- onError

 如果调用了这个方法说明出现了异常,此时也不会再调用`onNext`和`onCompleted`

- onCompleted

 在调用`onNext`后如果没有异常,最后会调用到这里,完成最后的逻辑,资源关闭释放之类的逻辑

- onSubscribe(可选)

 表明被观察者(Observable)已经准备好接受 observer 的请求了(在`背压`中有用到)

3. Operators — 操作符

对 Observable 做数据处理,filter,map,buffer,window 等

4. Backpressure — 背压

1. 定义:指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略,即`流控策略`

2. 适用范围:并不是 rxjava 的所有实现都支持背压

### Hystrix 中常见的 RxJava 套路

# Reference

- [RxJava——响应式编程](https://www.cnblogs.com/webor2006/p/7499622.html)
- [浅谈 RxJava 与多线程并发](http://zjutkz.net/2017/02/09/浅谈RxJava与多线程并发/)
