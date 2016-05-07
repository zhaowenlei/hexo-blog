---
title: java技术
date: 2016-05-08 00:13:49
tags: [技术分享标签]
category: [list和map之间的转换]
---

##list提取一个值为list:
```java
List<Long> storeIds=sysUserInviteStores.stream().map(SysUserInviteStore::getStoreId).collect(Collectors.toList());
```

list转map：value为对象:
```java
list.stream().collect(Collectors.toMap(MerchantParm::getKey,(o)->o));
```