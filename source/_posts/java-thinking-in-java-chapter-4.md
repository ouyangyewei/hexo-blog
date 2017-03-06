---
title: 《Thinking In Java》学习笔记：循环与值拷贝
categories:
- java
tags:
- java
date: 2017-02-01
toc: false
---

### `Foreach`与值拷贝

```java
char[] array = "boss".toCharArray();
for (char c : array) {
    if (c == 'o') {
        c = 'a';
    }
}
System.out.println(array);

/* Output:
boss
*///:~
```

实际上，上面的程序等价于：

```java
char[] array = "boss".toCharArray();
for (int i=0; i<array.length; ++i) {
    char c = array[i];
    if (c == 'o') {
        c = 'a';
    }
}
System.out.println(array);

/* Output:
boss
*///:~
```

在这里，变量`c`只是迭代器的局部变量，是对`array`的值拷贝，并非引用，因此改变值并不会改变数组
