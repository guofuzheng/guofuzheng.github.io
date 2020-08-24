---
layout:     post                    # 使用的布局（不需要改）
title:      SpringBoot学习记录               # 标题 
subtitle:   SpringBoot学习记录 #副标题
date:       2020-07-06             # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - SpringBoot
    - Java
---

### 1 页面无法访问404

如果页面都放在Resources中，但是页面就是一直无法访问，有可能是没有引入模板引擎。
如果引入模板引擎后，还是页面报404的错误，页面无法访问，有可能是静态资源被拦截了。
在application.properties中加入
```java
spring.resources.static-locations=classpath:templates(classpath:static)
```

### 2 @Controller和@RestController的区别

首先，@Controller和@RestController的共同点是：用来表示Spring某个类是否可以接受HTTP请求。

他俩的区别是：@Controller标识一个类是Spring MVC Controller处理器。@RestController是@Controller和@RestController的结合体，两个标注合并起来的作用。如果只是使用@RestController注解@Controller的内容，那么页面不会跳转，只会返回引号中的内容。

### 3、关于Mapper的无法注入的问题

在编写数据库查询的Mapper时，需要在该类上加上注解@Mapper，告诉这个类是Mapper。
### 4、Mybatis遇到No constructor found in ....的解决方法
在使用mybatis时，偶尔遇到了“No constructor found in .....”的问题，根据问题的提示可以看出，应该是构造方法引起的异常，经测试，当引用的实体重构了构造方法之后就会出现这个问题，因为mybatis需要用到默认构造方法，明确一个默认构造方法即可解决。
错误代码：
```java
public class Employee {
    private Integer EmployeeId;
    private String LastName;
    private String email;
    private Integer gender;
    private Integer DepartmentId;
    public Employee(Integer employeeId, String lastName, String email, Integer gender, Integer departmentId) {
        EmployeeId = employeeId;
        LastName = lastName;
        this.email = email;
        this.gender = gender;
        DepartmentId = departmentId;
    }
}
```
正确代码：
```java
public class Employee {
    private Integer EmployeeId;
    private String LastName;
    private String email;
    private Integer gender;
    private Integer DepartmentId;
    public  Employee(){
        super();
    }
    public Employee(Integer employeeId, String lastName, String email, Integer gender, Integer departmentId) {
        EmployeeId = employeeId;
        LastName = lastName;
        this.email = email;
        this.gender = gender;
        DepartmentId = departmentId;
    }
```
### 5、@RestControllerAdvice和@ControllerAdvice的区别
我自己的实验结果是@ControllerAdvice会返回定制的错误的页面，但是@RestControllerAdvice则会返回JSON数据。应该是与@RestController和@Controller的区别差不多。
### 6、在处理消息时，监听和发送有可能会出现监听接收不到的情况
之前遇到这个问题，是因为发送的一个自己新建的实体类，发送过去但是接收不到。在加入自定义的序列化方法后，解决这个问题。
```java
package com.guo.springbootamqp.config;

import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyAMQPConfig {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```
### 7、class path contains multiple SLF4J bindings
这是因为依赖中有多个SLF4J，导致的问题，所以排除多余的依赖就可以了，之前遇到的是
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```
和
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

这两个冲突，去掉了log4j2就好了，可能后面还是会有问题，先这么记录。
### 8、在注册Service上启用缓存，并以UserName作为键值会出现同名的无法注册
这个问题我的想法是，在我操作的时候，是因为把数据库里的数据删掉了，然后缓存里面的没有删除，所以redis缓存就认为数据已经存在数据库里了，就不在执行数据库的操作，所以在注册时，数据库不会新增记录。
### 9、mybatis批量添加
xml写法：
```xml
<insert id="addUserBatch" useGeneratedKeys="true" parameterType="java.util.List">
        INSERT INTO userTable(userNickname,userPassword)
        VALUES
            <foreach collection="list" index="index" item="user" separator=",">
                (#{user.userNickname},#{user.userPassword})
            </foreach>
</insert>
```
> 这里的collection可以选择list或者collection，但是进行选择list。  

mapper写法：
```java
public Integer addUserBatch(List<UserEntity> userEntityList);
```
service写法：
```java
public Integer addUserBatch(List<UserEntity> userEntityList){
        Integer flag = userMapper.addUserBatch(userEntityList);
        return flag;
    }
```
controller写法：
```java
public Integer addUserBatch(@RequestBody List<UserEntity> userEntityList){
        Integer flag = userService.addUserBatch(userEntityList);
        return flag;
    }
```
其中，在使用postman发送请求的时候，填好请求路径，然后在body中选择raw，选择JSON格式，输入JSON数据：
```json
[
{
"userNickname":"zhangsan",
"userPassword":"123456"
},
{
"userNickname":"lisi",
"userPassword":"123456"
}
]
```
###  10、spring boot中JSON的创建与解析
可以使用fastjson这个包,关于创建一个对象的JSON,可以如下操作:
```java
UpdateGroupExternalEntity updateGroupExternalEntity = new UpdateGroupExternalEntity();
updateGroupExternalEntity.setBeforeUpdateGroupname("test1");
updateGroupExternalEntity.setAfterUpdateGroupname("test2");
String updateJSON = JSON.toJSONString(updateGroupExternalEntity);
```
关于解析JSON可以使用如下:
```java
JSONObject testObject = JSONObject.parseObject(test);
return testObject.getString("beforeUpdateGroupname");
```
### 11、关于Jaskson序列与反序列化Redis存取
```java
@PostMapping("listserialize")
    public String insertListSerialize() {
        ObjectMapper objectMapper = new ObjectMapper();
        List<String> stringList = new ArrayList<>();
        stringList.add("xulu");
        stringList.add("guofuzheng");
        stringList.add("zhangtao");
        stringList.add("yuanwei");
        stringList.add("zouping");
        stringList.add("shimengye");
        try{
            String test = objectMapper.writeValueAsString(stringList);
            redisTemplate.opsForValue().set("listserialize",test);
            return test;
        }catch (JsonProcessingException e){
            e.printStackTrace();
            return "";
        }
    }
@GetMapping("listdeserialize")
    public void getListSerialize(){
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            String json = (String) redisTemplate.opsForValue().get("listserialize");
            List<String> stringList = objectMapper.readValue(json, List.class);
            System.out.println(stringList);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
```
上面的是关于String类型的序列化与反序列化，存入redis中，如果存入的是实体对象的List，那么类似的就可以改为：
```java
    @PostMapping("/listserialize")
    public String insertListSerialize() {
        ObjectMapper objectMapper = new ObjectMapper();
        StudentEntity studentEntity1 = new StudentEntity(1,"guo",100.0);
        StudentEntity studentEntity2 = new StudentEntity(2,"sun",100.0);
        List<StudentEntity> studentEntities = new ArrayList<>();
        studentEntities.add(studentEntity1);
        studentEntities.add(studentEntity2);
        try{
            System.out.println(studentEntities);
            String test = objectMapper.writeValueAsString(studentEntities);
            redisTemplate.opsForValue().set("studentserialize",test);
            return test;
        }catch (JsonProcessingException e){
            e.printStackTrace();
            return "";
        }
    }
    @GetMapping("/listdeserialize")
    public void getListSerialize(){
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            String json = (String) redisTemplate.opsForValue().get("studentserialize");
            List<StudentEntity> stringList = objectMapper.readValue(json, List.class);
            System.out.println(stringList);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
```
### 12、关于使用jackson处理实体对象
如果需要在读取json后对，如果json是List<class>，需要对List中的对象进行操作，那么就需要下面的写法。
```java
public void test(){
        StudentEntity studentEntity1 = new StudentEntity(1,"guo",100.0);
        StudentEntity studentEntity2 = new StudentEntity(2,"sun",100.0);
        List<StudentEntity> studentEntities = new ArrayList<>();
        studentEntities.add(studentEntity1);
        studentEntities.add(studentEntity2);
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            String studentEntitiesJson = objectMapper.writeValueAsString(studentEntities);
            List<StudentEntity> stringList = objectMapper.readValue(studentEntitiesJson,
                    objectMapper.getTypeFactory().constructCollectionType(List.class, StudentEntity.class));
            System.out.println("listener");
            System.out.println(stringList);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
```
`在使用jackson处理实体对象时，需要该实体类中有无参构造器，否则无法完成正常的转换。`不然会报类似下面的错误：
```
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `com.guo.redis.Entity.StudentEntity` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (String)"[{"studentId":1,"studentName":"guo","studentScore":100.0},{"studentId":2,"studentName":"sun","studentScore":100.0}]"; line: 1, column: 3] (through reference chain: java.util.ArrayList[0])
```
### 13、关于RabbitMQ的使用
1. 先设置MessageConverter
    ```java
    package com.guo.redis.Config;

    import org.springframework.amqp.rabbit.connection.ConnectionFactory;
    import org.springframework.amqp.rabbit.core.RabbitTemplate;
    import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class MyAMQPConfig {
        @Bean
        public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
            RabbitTemplate template = new RabbitTemplate(connectionFactory);
            template.setMessageConverter(new Jackson2JsonMessageConverter());
            return template;
        }
    }
    ```
2. 设置发送者
```java
@RestController
public class RabbitMqController {
    @Autowired
    RabbitTemplate rabbitTemplate;
    @GetMapping("/rabbit/send")
    public void sendGuo(){
        StudentEntity studentEntity1 = new StudentEntity(1,"guo",100.0);
        StudentEntity studentEntity2 = new StudentEntity(2,"sun",100.0);
        List<StudentEntity> studentEntities = new ArrayList<>();
        studentEntities.add(studentEntity1);
        studentEntities.add(studentEntity2);
        rabbitTemplate.convertAndSend("exchange.topic","guo",studentEntities);
    }
}
```
3. 搞定消费者
```java
@Component
public class RabbitMQListener {
    @RabbitListener(queues = "guo")
    public void getGuo(Message message)  {
        ObjectMapper objectMapper = new ObjectMapper();
        try {
            List<StudentEntity> studentEntityList = objectMapper.readValue(new String(message.getBody()), objectMapper.getTypeFactory().constructCollectionType(List.class, StudentEntity.class));
            System.out.println("queue:guo->"+  studentEntityList);
        }catch (JsonProcessingException e){
            e.printStackTrace();
        }
    }
}
```
message.getBody()是Byte[]数组类型的，在形参里设置String message也可以直接接受数据就是像这样`public void getGuo(String message){}`   
### 14、关于Swagger2的使用   
在安装Swagger后，需要相关配置，同时还要防止拦截器拦截页面造成404，配置类可以类似于下面的写法：   

```java
package com.guo.mypost7.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class MySwaggerConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/swagger-ui.html").addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**").addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
    @Bean
    public Docket MyPostApi(){
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(new ApiInfoBuilder()
                        .title("Post社交平台接口说明文档")
                        .description("这是Post社交平台接口说明文档的描述")
                        .contact(new Contact("guofuzheng","guofuzheng.gitee.io","guofuzheng@foxmail.com"))
                        .version("版本号：V1.0")
                        .build())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.guo.mypost7"))
                .paths(PathSelectors.any())
                .build();
    }
}

```
其中，`addResourceHandlers`是为了防止拦截器拦截swagger页面配置的。因此配置类需要继承WebMvcConfigurationSupport。   
`这里面还有个问题，如果在Spring boot1.5.12中WebMvcConfigurerAdapter()配置了拦截器会导致/无法访问，原因不详`   
原因已经找到了：在定义Swagger的时候，可以不需要增加资源路径。就是像这样写：   

```java
@Configuration
@EnableSwagger2
public class MySwaggerConfig{
    @Bean
    public Docket MyPostApi(){
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(new ApiInfoBuilder()
                        .title("Post社交平台接口说明文档")
                        .description("这是Post社交平台接口说明文档的描述")
                        .contact(new Contact("guofuzheng","guofuzheng.gitee.io","guofuzheng@foxmail.com"))
                        .version("版本号：V1.0")
                        .build())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.guo.mypost7"))
                .paths(PathSelectors.any())
                .build();
    }
}
```
### 15、关于Mybatis Plus中主键设置空插入不了
在使用PostgreSQL和Mybatis Plus结合时，出现了不设置表主键无法插入的情况。这种问题可以在表实体类中的`@TableId()`设置，具体如下：
```java
@TableId(type = IdType.AUTO) 主键自增 数据库中需要设置主键自增
private Long id;
@TableId(type = IdType.NONE) 默认 跟随全局策略走
private Long id;
@TableId(type = IdType.UUID) UUID类型主键
private Long id;
@TableId(type = IdType.ID_WORKER) 数值类型  数据库中也必须是数值类型 否则会报错
private Long id;
@TableId(type = IdType.ID_WORKER_STR) 字符串类型   数据库也要保证一样字符类型
private Long id;
@TableId(type = IdType.INPUT) 用户自定义了  数据类型和数据库保持一致就行
private Long id;
```
同时也可以在`application.yml`中设置：
```yml
mybatis-plus:
  mapper-locations:
    - com/mp/mapper/*
  global-config:
    db-config:
      id-type: uuid/none/input/id_worker/id_worker_str/auto   表示全局主键都采用该策略
```
