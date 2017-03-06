---
title: 《Thinking In Java》学习笔记：static与构造器顺序
categories:
- java
tags:
- java
date: 2017-02-02
toc: false
---

### `static`与构造器顺序
```java
class A {
    A(int n) {
        System.out.println("A(" + n + ")");
    }
}

class B {
    B(int n) {
        System.out.println("B(" + n + ")");
    }
}

public class TestStatic {

    static A a1 = new A(1);
    B b1 = new B(1);
    static A a2 = new A(2);
    B b2 = new B(2);
    A a3;
    A a4;

    static {
        System.out.println("Inside static block.");
    }

    TestStatic() {
        System.out.println("Inside Constructor.");
        a3 = new A(3);
        a4 = new A(4);
    }

    public static void main(String[] args) {
        System.out.println("Creating TestStatic");
        new TestStatic();
    }

    static A a5 = new A(5);
    static A a6 = new A(6);
}
/* Output:
A(1)
A(2)
Inside static block.
A(5)
A(6)
Creating TestStatic
B(1)
B(2)
Inside Constructor.
A(3)
A(4)
*///:~
```

从上面的程序中，可以发现：
> 初始化的顺序是先*静态对象*，然后是*“非静态”对象*
