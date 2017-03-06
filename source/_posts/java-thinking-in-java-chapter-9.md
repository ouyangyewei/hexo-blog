---
title: 《Thinking In Java》学习笔记：抽象类与接口
categories:
- java
tags:
- java
date: 2017-02-05
toc: true
---

### 抽象类
用于捕捉子类的共有特性，它不能被实例化，只能用作子类的超类。抽象类是被用来创建继承层级里子类的模板。
```java
public abstract class AbstractClass {

    public AbstractClass() {
        // do sth
    }

    abstract void af1();
    public abstract void af2();
    protected abstract void af3();

    // ! abstract方法只能是非private的
    // private abstract void af4();

    void f1() {}
    public void f2() {}
    protected void f3() {}
    private void f4() {}
}
```


### 接口
是抽象方法的集合。只能被实现，本身不能做任何事情。若一个类实现了某个接口，则它就继承了这个接口的所有抽象方法。以JDK中的CharSequence.java为例
```java
public interface CharSequence {

    int length();

    char charAt(int index);

    CharSequence subSequence(int start, int end);

    public String toString();
}
```

#### 接口的特性
* 接口定义中的所有方法，都是public，隐式的abstract方法
* 接口定义中的域，都隐式是public static final
* 接口可嵌套在类和其它接口中

#### 使用接口的核心原因
* 能向上转型为多个基类型
* 防止创建基类的对象

### 抽象类与接口
| 项 | 抽象类 | 接口 |
| :--- | :--- | :--- |
| 构造器 | 可以有 | × |
| 实例化 | × | × |
| 属性 | 可以有。public，protected，default和private均可 | 可以有。只能是public static final |
| 方法 | 有private方法，非abstract方法可以实现 | 均是public，隐式的abstract方法 |
| 设计理念 | is-a关系（继承） | like-a关系（组合） |
| main方法 | 可以有，能运行 | 无 |
| 添加新方法 | 往抽象类添加新方法，可以提供默认实现，不需要改动现有代码 | 往接口添加新方法，必须改变实现该接口的类 |

### 何时使用抽象类和接口
* 如果你拥有一些方法并希望它们中的一些有默认实现，则使用抽象类
* 如果你想实现多重继承，那么必须使用接口。因为Java不支持多重继承，子类不能继承多个类，但能实现多个接口
* 如果基础功能不断改变，那么就需要使用抽象类。如果不断改变基础功能并使用接口，那么你就需要改变实现该接口的所有子类

### 参考材料
[Java抽象类与接口的区别][1]
[接口和抽象类有什么区别][2]

[1]: http://www.importnew.com/12399.html
[2]: http://blog.csdn.net/ooppookid/article/details/51173179
