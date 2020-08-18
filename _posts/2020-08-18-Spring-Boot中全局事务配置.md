---
layout:     post                    # 使用的布局（不需要改）
title:      Spring Boot中全局事务配置               # 标题 
subtitle:   Spring Boot中全局事务配置 #副标题
date:       2020-08-18              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-miui-ux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Spring Boot

---

在Spring Boot中需要建立全局事务来管理整个项目的事务操作，在不用一个个service上写注解的情况下，方便了整体项目的管理。

1. 在application.properties中添加：

```xml
transactional.method.required=save*,delete*,update*,exec*,set*,insert*,add*,imp*
transactional.method.readOnly=get*,query*,find*,select*,list*,is*,count*
```

2. 全局事务管理配置类：

```java
@Component
@Aspect
public class MyTransactionConfig {

    @Autowired
    private TransactionManager transactionManager;

    //指定事务处理范围
    private static final String POINTCUT_EXPRESSION = "execution(* com.guo.mypost.serviceImpl..*(..))";

    @Value("#{'${transactional.method.required}'.split(',')}")
    private List<String> requiredList;

    @Value("#{'${transactional.method.readOnly}'.split(',')}")
    private List<String> readOnlyList;


    @Bean
    public TransactionInterceptor txAdvice() {

        DefaultTransactionAttribute txAttr_REQUIRED = new DefaultTransactionAttribute();
        txAttr_REQUIRED.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

        DefaultTransactionAttribute txAttr_REQUIRED_READONLY = new DefaultTransactionAttribute();
        txAttr_REQUIRED_READONLY.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        txAttr_REQUIRED_READONLY.setReadOnly(true);

        NameMatchTransactionAttributeSource source = new NameMatchTransactionAttributeSource();

        Map<String,TransactionAttribute> nameMap = new LinkedHashMap<>();
        requiredList.forEach(r->nameMap.put(r,txAttr_REQUIRED));
        readOnlyList.forEach(r->nameMap.put(r,txAttr_REQUIRED_READONLY));
        source.setNameMap(nameMap);

        return new TransactionInterceptor(transactionManager, source);
    }

    @Bean
    public Advisor txAdviceAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression(POINTCUT_EXPRESSION);
        return new DefaultPointcutAdvisor(pointcut, this.txAdvice());
    }
}
```

> 需要注意的是，在指定事务处理范围的时候一定要指定具体实现类，不能指向service接口 。   

3. 可以通过如下方法测试：

```java
@Service
public class UserService {
 
    @Autowired
    private UserMapper userMapper;
 
    public void insertTest(){
        List<User> users = userMapper.selectList(new QueryWrapper<User>().eq("rid",3));
        User user = new User();
        user.setAccount("asdasd");
        user.setPassword("asasa");
        user.setRid(3);
        user.setUid(100);
        userMapper.insert(user);
        int a = 1/0;
    }
}
```

