---
layout:     post                    # 使用的布局（不需要改）
title:      webservice               # 标题 
subtitle:   webservice #副标题
date:       2020-10-16              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-miui-ux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - webservice   
    
---

webservice在我的理解看来就是方便两个系统或者模块调用接口，比如，如果两个系统分别运行在不同的服务器上，两者要调用接口，就需要两者在同一个局域网下（注册到同一个服务注册中心），而使用webservice则不需要。

### POM

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.1.17.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
            <version>3.4.0</version>
        </dependency>
        <dependency>
            <groupId>org.codehaus.jettison</groupId>
            <artifactId>jettison</artifactId>
            <version>1.3.7</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

> 主要是cxf-spring-boot-starter-jaxws这个包。

### 实体对象

```java
package org.example.entity;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class User {
    private String name;
    private String id;
}
```

### 编写接口

```java
package org.example.service;

import org.example.entity.User;

import javax.jws.WebMethod;
import javax.jws.WebService;


@WebService(name = "HelloService",targetNamespace = "entity.example.org")
public interface HelloService {
    /**
     * webservice
     * @param name 名称
     * @return string
     */
    @WebMethod
    User hello(String name);

}

```

> 这里有几个注意点，@WebService和@WebMethod两个注解不能忘记，同时，@WebService中的参数也不能忘记，关于targetNamespace中的，我的理解是返回参数的包空间，如果返回的是实体对象，而不是简单的字符串，那么就需要指向实体对象的包空间，就是entity.example.org，如果指向service.example.org，在结果输出的时候只能输出结果的内存那种数据，不能完整的显示实体对象的数据，而且不能转换类型。  

### 方法实现

```java
package org.example.service.impl;

import org.example.entity.User;
import org.example.service.HelloService;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.PathVariable;

import javax.jws.WebService;
import java.awt.*;


@Component
@WebService(name = "HelloService", targetNamespace = "entity.example.org",
        endpointInterface = "org.example.service.HelloService")
public class HelloServiceImpl implements HelloService {

    public User hello(String name) {
        User user = User.builder().id("2222").name(name).build();
        System.out.println(name);
        return user;
    }
}

```

> 这里的注意点主要是在@WebService中的name和接口中的一致，targetNamespace也需要一致，enpointInterface则需要指向实现的接口。

### 发布

在写完接口和实现方法后就需要把接口发布出去，就需要配置。

```java
package org.example.config;

import org.apache.cxf.Bus;
import org.apache.cxf.jaxws.EndpointImpl;
import org.example.service.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.xml.ws.Endpoint;

@Configuration
public class WebConfig {

    @Autowired
    private Bus bus;
    @Autowired
    private HelloService helloService;
    @Bean
    public Endpoint endpoint(){
        EndpointImpl endpoint = new EndpointImpl(bus,helloService);
        endpoint.publish("/ws/api");
        return endpoint;
    }

}

```

> 这里发布的就是之前定义的接口，HelloService。endpoint.publish("/ws/api")就是发布的路径，就是调用的路径。在发布完运行后，可以在浏览器查验http://localhost:8080/services/ws/api?wsdl 看到是否发布成功。

### 调用

在发布完之后，自然是需要调用的，在我的理解中，上面的操作都是服务提供方需要的操作，而服务调用方则只需要下面的操作。

```java
import org.apache.cxf.endpoint.Client;
import org.apache.cxf.jaxws.endpoint.dynamic.JaxWsDynamicClientFactory;
import org.example.entity.User;
import org.springframework.web.bind.annotation.RestController;

public class Test {
    @org.junit.Test
    public void test(){
        JaxWsDynamicClientFactory dcf = JaxWsDynamicClientFactory.newInstance();
        Client client = dcf.createClient("http://localhost:8080/services/ws/api?wsdl");

        try{
            Object[] objects = client.invoke("hello","guo");
            System.out.println(objects[0].toString());
        }catch (final Exception e){
            e.printStackTrace();
        }
    }
}

```

> JaxWsDynamicClientFactory在我看来主要是定义一个工厂，然后拆功能键连接，最后在invoke中调用方法，invoke(方法名，参数)。client.invoke()返回的就是一个对象数组。

至此调用完成。

补一个调用结果

服务提供方

```

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::       (v2.1.17.RELEASE)

2020-10-16 10:29:41.405  INFO 29076 --- [           main] org.example.Webservice                   : Starting Webservice on LAPTOP-28UR8PG3 with PID 29076 (C:\guofuzheng\Document\webservice\target\classes started by guofu in C:\guofuzheng\Document\webservice)
2020-10-16 10:29:41.408  INFO 29076 --- [           main] org.example.Webservice                   : No active profile set, falling back to default profiles: default
2020-10-16 10:29:42.263  INFO 29076 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-10-16 10:29:42.286  INFO 29076 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-10-16 10:29:42.286  INFO 29076 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.38]
2020-10-16 10:29:42.402  INFO 29076 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-10-16 10:29:42.403  INFO 29076 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 962 ms
2020-10-16 10:29:42.654  INFO 29076 --- [           main] o.a.c.w.s.f.ReflectionServiceFactoryBean : Creating Service {entity.example.org}HelloServiceImplService from class org.example.service.HelloService
2020-10-16 10:29:43.011  INFO 29076 --- [           main] org.apache.cxf.endpoint.ServerImpl       : Setting the server's publish address to be /ws/api
2020-10-16 10:29:43.111  INFO 29076 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-10-16 10:29:43.226  INFO 29076 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-10-16 10:29:43.228  INFO 29076 --- [           main] org.example.Webservice                   : Started Webservice in 2.098 seconds (JVM running for 3.793)
guo
```

服务调用方

```
10:30:07.173 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.binding.soap.interceptor.ReadHeadersInterceptor@60b85ba1
10:30:07.182 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.binding.soap.interceptor.SoapActionInInterceptor@3d829787
10:30:07.182 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.binding.soap.interceptor.StartBodyInterceptor@492fc69e
10:30:07.182 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.binding.soap.interceptor.MustUnderstandInterceptor@2fb68ec6
10:30:07.182 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.binding.soap.interceptor.CheckFaultInterceptor@117632cf
10:30:07.182 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.wsdl.interceptors.DocLiteralInInterceptor@71652c98
10:30:07.197 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.binding.soap.interceptor.SoapHeaderInterceptor@51bde877
10:30:07.197 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.jaxws.interceptors.WrapperClassInInterceptor@6b5f8707
10:30:07.199 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.interceptor.StaxInEndingInterceptor@39fc6b2c
10:30:07.199 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.jaxws.interceptors.SwAInInterceptor@5a12c728
10:30:07.199 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.jaxws.interceptors.HolderInInterceptor@772485dd
10:30:07.199 [main] DEBUG org.apache.cxf.phase.PhaseInterceptorChain - Invoking handleMessage on interceptor org.apache.cxf.ws.policy.PolicyVerificationInInterceptor@24528a25
10:30:07.199 [main] DEBUG org.apache.cxf.ws.policy.PolicyVerificationInInterceptor - Verified policies for inbound message.
User(name=guo, id=2222)

Process finished with exit code 0

```

