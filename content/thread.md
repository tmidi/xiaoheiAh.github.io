---
title: "thread"
date: 2019-08-15T13:19:18+08:00
draft: true
---



# ThreadPoolExecutor

> 对于业务逻辑复杂,影响响应时间的操作,可以通过线程池进行异步响应,减缓响应时间
> 源码里的注释一般就会很清晰的介绍该类了,看起来还是很方便的,主要就是词汇量不够还得划词翻译

## 主要参数
- corePoolSize 
  线程池的大小
- maximumPoolSize
  最大线程池大小
  如果线程池内线程数量小于corePoolSize,即使线程池中有闲置的县城可以调,还是会创建新线程,如果线程池内线程>corePoolSize但 <  maximumPoolSize,就会将线程放到workQueue里,queue满才会创建新线程,如果设置corePoolSize=maximumPoolSize,就是一个固定
  大小的线程池
- keepAliveTime
  当线程池>corePoolSize时,大于keepAliveTime后会终止多余的闲置线程,回收资源,设置为Long.MAX_VALUE则会防止中止
- unit
  时间度量单位
- workQueue
  线程池>corePoolSize时,会将后续的task存入queue中,而不是继续创建新thread,当queue存满了之后,如果线程数没有达到maximumPoolSize
  时会继续创建新线程,大于maximumPoolSize时会执行拒绝策略
- threadFactory
  threadFactory可以不创建,不创建则会使用默认的factory,自定义时,可以设置线程的name,group,优先级,守护线程状态等等
- handler
  拒绝策略


博文参考:
- [线程池大小设置与BlockingQueue的三种实现区别](http://dongxuan.iteye.com/blog/901689)
- 
