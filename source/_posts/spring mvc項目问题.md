---
title: spring maven 依赖引发的问题
date: 2017-04-02 12:24:35
category: [spring maven]
tags: [spring maven]
---

今天做搞spring项目，因为要写jsp，所以maven引入依赖，结果报了一大堆错误，网上搜了好多资料都没搞定，后来干脆就不弄了，睡了一觉，然后又来弄了一下，结果无意中又看了一遍文章。奇迹般的搞定了，希望和一样遇到该错误的人不要走弯路。
错误如下：
![](/imgs/301901220093456.png)
遗憾的是出现 <font color=red size=3>color=Unable to compile class for JSP</font> 的错误.
　　网上寻找资料后知晓, 该问题由maven引入的servlet-api和tomcat自带的servlet-api版本发生了冲突.
　　解决方案如下:
　　maven的pom文件中, 把<font color=red size=3>color=servlet-api</font>依赖项的<font color=red size=3>color=scope</font>改为<font color=red size=3>color=provided</font>(即参与compile和test, 但不参与package).

```xml
<dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.0</version>
    </dependency>

    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
    </dependency>
```
改为
```xml
<dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.0</version>
      <scope>provided</scope>
    </dependency>

    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>
```