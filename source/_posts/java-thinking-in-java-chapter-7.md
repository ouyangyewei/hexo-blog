---
title: 《Thinking In Java》学习笔记：final与private关键字
categories:
- java
tags:
- java
date: 2017-02-04
toc: true
---

### final关键字
* 对于基本类型，final使其数值恒定不变
* 对于对象引用，final使其引用恒定不变（一旦引用被初始化指向一个对象，就无法再把它指向另一个对象，然而，对象其自身是可以修改的）

```java
class Value {
    int i;

    Value(int i) {
        this.i = i;
    }
}

public class TestFinal {
    private static final Value v1 = new Value(1);

    public static void main(String[] args) {
        v1.i++;
        System.out.println(v1.i);
    }
}
/* Output:
2
*///:~
```
> 引用`v1`在初始化时已经指向对象`new Value(1)`，就无法再指向其它对象，然而对象本身是可以改变的，它修改了成员变量`i`

### final类
* 不能被继承
* 成员变量并不隐式为final
* 所有方法都隐式指定为final，因此无法覆盖

### final与private关键字
* final方法：子类不可重写，只能调用
* private方法：本身是final方法，但是不能被子类继承，不能被子类重写，不能被子类调用

```java
public class Derived extends Base {
    public void func() {
        System.out.println("Derived.func");
    }
}

class Base {
    // 若方法改成非private，则会被子类同名方法覆盖
    private void func() {
        System.out.println("Base.func");
    }

    public static void main(String[] args) {
        Base obj = new Derived();

        // 由于private被自动认为是final方法，在子类不能被覆盖
        obj.func();
    }
}
/*
Base.func
*///:~
```
