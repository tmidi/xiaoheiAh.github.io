---
title: "rmi"
date: 2019-08-15T13:19:18+08:00
draft: true
---



# RMI笔记

> 环境: springboot 2.02

## RMI介绍
先说RPC,RPC是`远程过程调用 Remote Procedure Call`,从表面看,RPC类类似于调用一个本地对象的一个方法。
RMI即远程方法调用(Remote Method Invocation),通俗讲就是像在本地调用自己写的接口一样，是RPC模型的一种表现形式。还有些其他的RPC模型，类似于Hession,Burlp,
HTTP invoker,JAX—RPC。
## spring整合原始RMI调用实现
通常的调用方式为客户端向代理发起调用,代理代表客户端与远程服务进行通信,由代理负责处理连接的细节并向远程服务发起调用.调用远程服务时总是会发生`RemoteException`,代理会处理完异常同时再抛出`RemoteAccessException`.
Spring通过`RemoteExporter`将bean方法配置为远程服务,默认会绑定到1099端口并在该端口上启动一个`注册表`,如果端口被占用则会寻找合适的端口绑定.作为服务的bean最好实现`Serializable`接口.
代码如下:
```java
//新建service服务并实现
package com.springinaction.melon.rmi.service;

import java.net.UnknownHostException;

public interface HelloWorldRMIService {
    public void sayHelloWorld() throws UnknownHostException;
}

//实现HelloWorldRMIService
package com.springinaction.melon.rmi.service;

import org.springframework.stereotype.Service;

import java.net.Inet4Address;
import java.net.UnknownHostException;
@Service("helloWorldRMIService")
public class HelloWorldRMIServiceImpl implements HelloWorldRMIService {
    @Override
    public void sayHelloWorld() throws UnknownHostException {
        System.out.println("hello world from --> " + Inet4Address.getLocalHost());
    }
}
//通过RmiExporter配置服务并通过RmiProxyFactoryBean暴露服务
package com.springinaction.melon.rmi.config;

import com.springinaction.melon.rmi.service.HelloWorldRMIService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.remoting.rmi.RmiProxyFactoryBean;
import org.springframework.remoting.rmi.RmiServiceExporter;

@Configuration
public class RMIConfiguration {
    @Bean
    public RmiServiceExporter rmiExporter(HelloWorldRMIService helloWorldRMIService) {
        RmiServiceExporter rmiServiceExporter = new RmiServiceExporter();
        rmiServiceExporter.setService(helloWorldRMIService);
        rmiServiceExporter.setServiceName("HelloWorldRMIService");
        rmiServiceExporter.setServiceInterface(HelloWorldRMIService.class);
        //可以指定绑定host和port
        rmiServiceExporter.setRegistryHost("rmi.helloworld.com");
        rmiServiceExporter.setRegistryPort(1199);
        return rmiServiceExporter;
    }
    @Bean
    public RmiProxyFactoryBean helloWorldService() {
        RmiProxyFactoryBean rmiProxy = new RmiProxyFactoryBean();
        rmiProxy.setServiceUrl("rmi://localhost/HelloWorldRMIService");
        rmiProxy.setServiceInterface(HelloWorldRMIService.class);
        return rmiProxy;
    }
}
//模拟远程调用
package com.springinaction.melon;

import com.springinaction.melon.rmi.service.HelloWorldRMIService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MelonApplication implements CommandLineRunner {

    @Autowired
    HelloWorldRMIService helloWorldRMIService;

    @Value("melon.zhao")
    String name;

    public static void main(String[] args)  {
        SpringApplication.run(MelonApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println(args.length);
        helloWorldRMIService.sayHelloWorld();
        System.out.println(name);
    }
}

```


