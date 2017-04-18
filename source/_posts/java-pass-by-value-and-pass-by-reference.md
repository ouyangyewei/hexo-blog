---
title: Java 值传递与引用传递
categories:
- java
tags:
- java
date: 2017-03-13
toc: true
---

> Java在传递参数时，是通过值传递，而不是通过引用传递的

```java
public class Demo {
    public static void func_1(Dog a) {
        Dog b = new Dog("dog_b");
        a = b;
    }

    public static void func_2(Dog a) {
        a.setName("dog_a");
    }

    public static void main(String[] args) {
        Dog dog = new Dog("dog");
        func_1(dog);
        func_2(dog);
    }
}
```
其中类Dog如下：
![img_001][1]

---

下面我将通过结合代码与字节码来一步一步解释：

### main函数
```java
33  public static void main(String[] args) {
34      Dog dog = new Dog("dog");
35      func_1(dog);
36      func_2(dog);
37  }
```

```
public static void main(java.lang.String[]);
  descriptor: ([Ljava/lang/String;)V
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=3, locals=2, args_size=1
       0: new           #2   // 创建com/baidu/stack/Dog对象，并将引用压入操作栈
       3: dup                // 复制栈顶引用并将复制值压入操作栈
       4: ldc           #7   // 将字符串"dog"从常量池推送至操作数栈
       6: invokespecial #4   // 调用com/baidu/stack/Dog类的带参构造函数，初始化对象
       9: astore_1           // 将操作栈顶的对象引用存储到局部变量表slot 1（即dog）中
      10: aload_1            // 将局部变量中的对象引用dog加载到操作栈顶
      11: invokestatic  #8   // 调用方法func_1
      14: aload_1            // 将局部变量中的对象引用dog加载到操作栈顶
      15: invokestatic  #9   // 调用方法func_2
      18: return
    LineNumberTable:
      line 34: 0
      line 35: 10
      line 36: 14
      line 37: 18
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      19     0  args   [Ljava/lang/String;
         10       9     1   dog   Lcom/baidu/stack/Dog;
```

![img_002][2]

### 调用func_1函数
```java
24  public static void func_1(Dog a) {
25      Dog b = new Dog("dog_b");
26      a = b;
27  }
```

```
public static void func_1(com.baidu.stack.Dog);
  descriptor: (Lcom/baidu/stack/Dog;)V
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=3, locals=2, args_size=1
       0: new           #2   // 创建com/baidu/stack/Dog对象，并将引用压入操作栈
       3: dup                // 复制栈顶引用并将复制值压入操作栈
       4: ldc           #3   // 将字符串"dog_b"从常量池推送至操作数栈
       6: invokespecial #4   // 调用com/baidu/stack/Dog类的带参构造函数，初始化对象
       9: astore_1           // 将操作栈顶的对象引用存储到局部变量表slot 1（即b）中
      10: aload_1            // 将局部变量表中的slot 1的引用值（即b）加载到操作栈顶
      11: astore_0           // 将操作栈顶的对象引用存储到局部变量表slot 0（即a）中，即a=b
      12: return
    LineNumberTable:
      line 25: 0
      line 26: 10
      line 27: 12
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      13     0     a   Lcom/baidu/stack/Dog;
         10       3     1     b   Lcom/baidu/stack/Dog;
```
从上面的字节码中，可以看到：

* 在定义方法时，引用a被声明和初始化为null
```java
public static void func_1(Dog a)
```
![img_003][3]

* 在调用方法时，引用a将指向引用dog指向的对象
```java
func_1(dog);
```
![img_004][4]

* 在声明引用b时，创建了一个新的Dog类型对象，引用b并指向新创建的对象
```java
Dog b = new Dog("dog_b");
```
![img_005][5]

* a=b，重新将a的引用指向引用b指向的对象
```java
a = b;
```
![img_006][6]

---

### 调用func_2函数
```java
29  public static void func_2(Dog a) {
30      a.setName("dog_a");
31  }
```

```
public static void func_2(com.baidu.stack.Dog);
  descriptor: (Lcom/baidu/stack/Dog;)V
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=2, locals=1, args_size=1
       0: aload_0            // 将局部变量表中的slot 0的引用值（即a）加载到操作栈顶
       1: ldc           #5   // 将字符串"dog_a"从常量池推送至操作数栈
       3: invokevirtual #6   // 调用方法com/baidu/stack/Dog.setName
       6: return
    LineNumberTable:
      line 30: 0
      line 31: 6
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       7     0     a   Lcom/baidu/stack/Dog;
```
从上面的字节码中，可以看到：

* 在调用方法时，引用a将指向dog指向的对象
```java
func_2(dog);
```
![img_004][4]

* 调用Dog.setName方法，将会改变引用a所指向对象的name
```java
a.setName("dog_a");
```
![img_007][7]

---

### 参考资料
[Is-Java-pass-by-reference-or-pass-by-value][8]
[Java-is-Pass-by-Value-Dammit][9]


[1]: http://ol3q0aw97.bkt.clouddn.com/blog/java-pass-by-value-and-pass-by-reference/img_001.png
[2]: http://ol3q0aw97.bkt.clouddn.com/blog/java-pass-by-value-and-pass-by-reference/img_002.png
[3]: http://ol3q0aw97.bkt.clouddn.com/blog/java-pass-by-value-and-pass-by-reference/img_003.png
[4]: http://ol3q0aw97.bkt.clouddn.com/blog/java-pass-by-value-and-pass-by-reference/img_004.png
[5]: http://ol3q0aw97.bkt.clouddn.com/blog/java-pass-by-value-and-pass-by-reference/img_005.png
[6]: http://ol3q0aw97.bkt.clouddn.com/blog/java-pass-by-value-and-pass-by-reference/img_006.png
[7]: http://ol3q0aw97.bkt.clouddn.com/blog/java-pass-by-value-and-pass-by-reference/img_007.png
[8]: http://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value
[9]: http://javadude.com/articles/passbyvalue.htm
