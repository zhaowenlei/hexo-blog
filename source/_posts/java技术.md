---
title: list和map之间的转换
date: 2016-05-08 00:13:49
category: [java技术]
tags: [java技术]
---

### list提取一个值为list:

```java
List<Long> storeIds=sysUserInviteStores.stream().map(SysUserInviteStore::getStoreId).collect(Collectors.toList());
```

## list转map：value为对象:

```java
list.stream().collect(Collectors.toMap(MerchantParm::getKey,(o)->o));
```

## list转map：value为单值:

```java
list.stream().collect(Collectors.toMap(MerchantParm::getKey, MerchantParm::getVal));
```

## 根据对象里面的id来分组

```java
Map<Long, List<SysUserRoleDTO>> sysUserRoleMap = sysUserRoleDTOs.stream().collect(Collectors.groupingBy(SysUserRoleDTO::getStoreId));
```
