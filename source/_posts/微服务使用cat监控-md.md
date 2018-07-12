---
title: 微服务使用cat监控.md
date: 2018-07-12 17:09:42
category: [java技术]
tags: [微服务使用cat监控]
---

1. 项目加入依赖的jar包


```xml
  <dependency>
           <groupId>com.aixuexi</groupId>
           <artifactId>thor-util</artifactId>
           <version>1.4.9.1-RELEASE</version>
   </dependency>
```

2. 加入cat的AOP:
```xml
    <bean id="catAop" class="com.aixuexi.thor.aop.CatAop"></bean>
    <aop:config>
        <aop:pointcut id="service" expression="execution(* com.aixuexi.underworld.service.*.*(..))"/>
        <aop:aspect ref="catAop">
            <aop:around method="around" pointcut-ref="service"/>
        </aop:aspect>
    </aop:config>
```

3. 在resources资源文件META-INF下，注意是src/main/resources/META-INF/文件夹， 而不是webapps下的那个META-INF,添加app.properties，如：app.name=inception

4. 在项目部署的服务器上创建/data/appdatas/cat/client.xml,此文件有OP控制,这里的Domain名字用来做开关，如果一台机器上部署了多个应用，可以指定把一个应用的监控关闭。注意这里的目录必须要有读写权限

```xml
   <config mode="client">
        <servers>
             <server ip="10.26.83.53" port="2280" />
        </servers>
        <domain id="inception" enabled="true"/>
   </config>
```