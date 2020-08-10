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
