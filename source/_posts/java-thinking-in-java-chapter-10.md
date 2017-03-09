---
title: 《Thinking In Java》学习笔记：内部类
categories:
- java
tags:
- java
date: 2017-02-08
toc: true
---

> 内部类的访问控制符可以使用public、protected、private，而外部类只能使用public和默认

### 成员内部类与静态内部类
直接定义在外部类内部（不在方法或代码块内部），它可以直接访问外部类的所有成员变量和方法（包括private）。它可以有各种修饰符，包括：public、protected、private、static、final和abstract，也可以是默认；

#### 成员内部类
对象级别（依赖于外部类对象）。如同外部类的一个成员变量，不能声明任何static成员
```java
class Outer {
    private int outerValue = 5;
    public void addOne() { ++this.outerValue; }

    class Inner {
        public void showOuterValue() {
            // 成员内部类能直接访问外部类的成员变量和方法
            // 通过Outer.this（可以省略不写），内部类能获取到外部类的成员变量和方法
            System.out.println(Outer.this.outerValue);
            Outer.this.addOne();
            System.out.println(Outer.this.outerValue);
        }
    }
}

public class Demo {
    public static void main(String[] args) {
        // 内部类对象依赖外部类对象，需要先生成外部类对象
        Outer.Inner inner = new Outer().new Inner();
        inner.showOuterValue();
    }
}/* Output:
5
6
*///:~
```
外部类想要访问内部类的成员变量和方法，需要通过内部类的对象来获取。

#### 静态内部类
类级别（不依赖外部类对象）。如同外部类的一个静态成员变量，它的对象与外部类对象间不存在依赖关系。
```java
class Outer {
    private int outerValue = 1;
    private static int outerStaticValue = 2;
    private void f1() { System.out.println("Outer.f1"); }
    private static void f2() { System.out.println("Outer.f2"); }
    static class Inner {
        public void showOuterValue() {
            // !: 静态内部类不能访问外部类的非静态成员变量和方法
            // System.out.println(Outer.outerValue);
            // Outer.f1();

            // 静态内部类只能访问外部类的静态成员变量和方法
            System.out.println(Outer.outerStaticValue);
            Outer.f2();
        }
    }
}
public class Demo {
    public static void main(String[] args) {
        // 静态内部类对象的创建，不需要依赖外部类的对象，可直接创建
        Outer.Inner inner = new Outer.Inner();
        inner.showOuterValue();
    }
}/* Output:
2
Outer.f2
*///:~
```
若静态内部类使用private修饰，则只能在外部类中访问

---

### 局部内部类
定义在代码块的类（比如方法和作用域内），它们只在定义它们的代码块中是可见的。
```java
class Outer {
    private String o1 = "o1";
    private static String o2 = "o2";
    private final String o3 = "o3";

    void func() {
        final String f1 = "f1";
        String f2 = "f2";

        class Inner {
            public void showValue() {
                System.out.println(o1);
                System.out.println(o2);
                System.out.println(o3);
                System.out.println(f1);

                // !: 局部内部类不能访问非final型局部变量
                // System.out.println(f2);
            }
        }

        new Inner().showValue();
    }
}

public class Demo {
    public static void main(String[] args) {
        new Outer().func();
    }
}/* Output:
o1
o2
o3
f1
*///:~
```
局部内部类只在定义的代码块中可见，因此没有public、private、protected和default修饰符，因为没有意义。局部内部类不能被static修饰，因为局部内部类不再是成员的位置，它只在定义的代码块中可见。

---

### 匿名内部类
匿名内部类是局部内部类的一种特殊形式，必须继承一个父类或实现一个接口。匿名内部类很重要的一个作用就是缩减代码量。

不使用匿名内部类来实现抽象方法：
```java
interface Cycle { void ride(); }
interface CycleFactory { Cycle getCycle(); }

class Bicycle implements Cycle {
    @Override
    public void ride() {
        System.out.println("Bicycle");
    }
}

class Unicycle implements Cycle {
    @Override
    public void ride() {
        System.out.println("Unicycle");
    }
}

class BicycleFactory implements CycleFactory {
    @Override
    public Cycle getCycle() {
        return new Bicycle();
    }
}

class UnicycleFactory implements CycleFactory {
    @Override
    public Cycle getCycle() {
        return new Unicycle();
    }
}

public class Cycles {
    public static void main(String[] args) {
        CycleFactory[] cycleFactories = {
                new BicycleFactory(),
                new UnicycleFactory()
        };
        for (CycleFactory factory : cycleFactories) {
            factory.getCycle().ride();
        }
    }
}
```
可见，每新增一种Cycle工厂，都要先实现CycleFactory接口，然后实现出对象，再向上转型为CycleFactory引用，过程太过繁琐，而且代码量会逐渐膨胀。但是使用匿名内部类的方式就可以很好解决：
```java
interface Cycle { void ride(); }
interface CycleFactory { Cycle getCycle(); }

class Bicycle implements Cycle {
    public static CycleFactory factory = new CycleFactory() {
        @Override
        public Cycle getCycle() {
            return new Bicycle();
        }
    };

    private Bicycle() {}

    @Override
    public void ride() { System.out.println("Bicycle"); }
}

class Unicycle implements Cycle {
    public static CycleFactory factory = new CycleFactory() {
        @Override
        public Cycle getCycle() {
            return new Unicycle();
        }
    };

    private Unicycle() {}

    @Override
    public void ride() { System.out.println("Unicycle"); }
}

public class Cycles {
    public static void main(String[] args) {
        CycleFactory[] cycleFactories = {
                Bicycle.factory,
                Unicycle.factory
        };
        for (CycleFactory factory : cycleFactories) {
            factory.getCycle().ride();
        }
    }
}
```
可见，匿名内部类实现了CycleFactory接口并在大括号中实现了抽象方法。

---

### 内部类标识符
#### 非匿名的内部类
```java
class Outer1 {
    private String o1 = "o1";
    private static String o2 = "o2";
    private final String o3 = "o3";

    void func() {
        final String f1 = "f1";
        String f2 = "f2";

        class Inner {
            public void showValue(String f3) {
                System.out.println(o1);
                System.out.println(o2);
                System.out.println(o3);
                System.out.println(f1);
                System.out.println(f3);
            }
        };

        new Inner().showValue("f3");
    }
}
```
编译后：
```
$ ll
total 12
-rw-r--r-- 1 root root 616 Mar  9 15:20 Demo1.java
-rw-r--r-- 1 root root 828 Mar  9 15:20 Outer1$1Inner.class
-rw-r--r-- 1 root root 862 Mar  9 15:20 Outer1.class
```

#### 匿名的内部类
```
interface Base { void showValue(String f3); }

class Outer {
    private String o1 = "o1";
    private static String o2 = "o2";
    private final String o3 = "o3";

    Base getBase() {
        final String f1 = "f1";
        String f2 = "f2";

        return new Base() {
            @Override
            public void showValue(String f3) {
                System.out.println(o1);
                System.out.println(o2);
                System.out.println(o3);
                System.out.println(f1);
                System.out.println(f3);
            }
        };
    }
}
```
编译后：
```
$ ll
total 20
-rw-r--r-- 1 root root 158 Mar  9 16:10 Base.class
-rw-r--r-- 1 root root 497 Mar  9 16:10 Demo.class
-rw-r--r-- 1 root root 901 Mar  9 15:02 Demo.java
-rw-r--r-- 1 root root 885 Mar  9 16:10 Outer$1.class
-rw-r--r-- 1 root root 820 Mar  9 16:10 Outer.class
```

---

### 疑问
<u>**1. 为什么局部内部类和匿名内部类只能访问final类型的局部变量？**</u>
* 方法中定义的局部变量的生命周期会随着方法调用结束而结束，而定义在方法或作用域中的局部内部类
或匿名内部类的对象的生命周期可能还没结束，那么在局部内部类或匿名内部类继续访问方法的局部变量，
就变得不可能。
* 局部变量声明为final类型，在编译期间值就会得到确定，并直接在局部内部类或匿名内部类中创建一个拷贝

### 参考资料
[Why-are-only-final-variables-accessible-in-anonymous-class][1]
[Cannot-refer-to-a-non-final-variable-inside-an-inner-class-defined-in-a-different-method][2]
[为什么匿名内部类和局部内部类只能访问final变量][3]

[1]: http://stackoverflow.com/questions/4732544/why-are-only-final-variables-accessible-in-anonymous-class
[2]: http://stackoverflow.com/questions/1299837/cannot-refer-to-a-non-final-variable-inside-an-inner-class-defined-in-a-differen
[3]: http://blog.csdn.net/zhaoyw2008/article/details/9565219
