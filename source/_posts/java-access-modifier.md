---
title: Java 修饰符访问权限
categories:
- java
tags:
- java
date: 2017-01-08
toc: false
---

### 修饰符访问权限列表

| 修饰符 | 当前类 | 同一包内 | 子孙类 | 其他包
| :--- | :--- | :--- | :--- | :--- |
| public | √ | √ | √ | √ |
| protected | √ | √ | √ | × |
| default | √ | √ | × | × |
| private | √ | × | × | × |

参考:https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html
