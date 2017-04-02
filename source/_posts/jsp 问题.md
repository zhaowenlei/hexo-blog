---
title: jsp 依赖问题
date: 2017-03-26 12:24:35
category: [jsp]
tags: [jsp]
---

今天搞了一下jsp，做了个例子，其中有一句out.write(),结果总是报红，百度了好长时间还是没有搞定，国内的网站真是坑啊，说什么的都有，总之没有说到点儿上的。
最后google了一下，搞定。
You'll have this problem with out, pageContext and jspContext because they use classes provided with the JSP api (not the servlet API).

To use them (if you're working with a maven project) add this dependency :
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.0</version>
</dependency>

```
If you have the problem with every implicit object (session, request, etc.) you should add the servlet api dependency too :
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
```