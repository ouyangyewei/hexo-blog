---
title: 《Thinking In Java》学习笔记：修饰符访问权限
categories:
- java
tags:
- java
date: 2017-02-03
toc: false
---

### 修饰符访问权限列表

| 修饰符 | 当前类 | 同一包内 | 子孙类 | 其他包
| :--- | :--- | :--- | :--- | :--- |
| public | √ | √ | √ | √ |
| protected | √ | √ | √ | × |
| default | √ | √ | × | × |
| private | √ | × | × | × |
