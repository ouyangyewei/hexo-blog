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

从字节码角度来分析
```java
public static void main(java.lang.String[]);
    Code:
       0: ldc           #2      // 将字符串常量boss，从常量池推送至栈顶
       2: invokevirtual #3      // 调用实例方法：java/lang/String.toCharArray()
       5: astore_1              //
       6: aload_1               //
       7: astore_2              // 赋值："boss" -> array
       8: aload_2               // 将 array 压入栈顶
       9: arraylength           // 计算数组长度，并压入栈顶
      10: istore_3              // 赋值：0 -> arrayLength
      11: iconst_0              // 将 0 压入栈顶
      12: istore        4       // 赋值：0 -> i
      14: iload         4       // 将 i 压入栈顶
      16: iload_3               // 将 arrayLength 压入栈顶
      17: if_icmpge     43      // 比较 i < arrayLength
      20: aload_2               // 将 array 压入栈
      21: iload         4       // 将 i 压入栈
      23: caload                // 将 array[i] 压入栈顶，实际上压入的是array[i]对应的ASCII值
      24: istore        5       // 赋值：array[i]的ASCII值 -> c
      26: iload         5       // 将 c 压入栈顶
      28: bipush        111     // 将字符o对应的ASCII值压入栈顶
      30: if_icmpne     37      // 比较 c == 'o'
      33: bipush        97      // 将字符a的ASCII值压入栈顶
      35: istore        5       // 赋值：'a' -> c
      37: iinc          4, 1    // 自增：++i
      40: goto          14
      43: return
```
可见，foreach的内部实现，也是增加了临时变量i和arrayLength，而其中的c只是局部变量（栈空间，使用完即不存在），取的array[i]只是值拷贝，并非引用，因此，实际上程序等同于如下：
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
