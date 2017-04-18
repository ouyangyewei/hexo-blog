---
title: Java String
categories:
- java
tags:
- java
date: 2017-04-01
toc: true
---

### String类
#### String类是不可变类
下面代码片段摘自java.lang.String
``` java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
```
可见，String类使用final关键字修饰，是不可变类，因此不可被继承；

#### 字符串的内存形态
String对象在内存中使用不可变的char型数组来存放字符串，因此String对象在创建后，其值就不可被改变（对于修改已存在的String对象的值，实际上是会创建一个新的String对象，可参考java.lang.String.concat方法的实现）

下面代码摘自java.lang.String
``` java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
```
下面代码摘自java.util.Arrays
``` java
public static char[] copyOf(char[] original, int newLength) {
    char[] copy = new char[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```
![java.lang.String.concat][2]

#### String中的"+"操作符
引用String类的注释
> The Java language provides special support for the string concatenation operator ( + ), and for conversion of other objects to strings. String concatenation is implemented through the StringBuilder(or StringBuffer) class and its append method. String conversions are implemented through the method toString, defined by Object and inherited by all classes in Java. For additional information on string concatenation and conversion, see Gosling, Joy, and Steele, The Java Language Specification.

Java中，String的"+"操作符，是通过StringBuilder（或StringBuffer）类和它的append方法实现的，下面以一段代码来证明：
``` java
public class Turtle {
    public static void main(String[] args) {
        String a = "a";
        String b = "b";
        String c = "c";
        String d = a + b + c;
    }
}
```
对应的字节码：
``` java
public static void main(java.lang.String[]);
  descriptor: ([Ljava/lang/String;)V
  flags: ACC_PUBLIC, ACC_STATIC
  Code:
    stack=2, locals=5, args_size=1
       0: ldc           #2  // String a
       2: astore_1
       3: ldc           #3  // String b
       5: astore_2
       6: ldc           #4  // String c
       8: astore_3
       9: new           #5  // class java/lang/StringBuilder
      12: dup
      13: invokespecial #6  // Method java/lang/StringBuilder."<init>":()V
      16: aload_1
      17: invokevirtual #7  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      20: aload_2
      21: invokevirtual #7  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      24: aload_3
      25: invokevirtual #7  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      28: invokevirtual #8  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      31: astore        4
      33: return
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      34     0  args   [Ljava/lang/String;
          3      31     1     a   Ljava/lang/String;
          6      28     2     b   Ljava/lang/String;
          9      25     3     c   Ljava/lang/String;
         33       1     4     d   Ljava/lang/String;
```
可见，"+"的操作，实际上会创建一个StringBuilder对象，先后调用append()方法，添加字符串"a"，"b"，"c"，最后调用toString()方法返回一个String对象

---

### 参考材料
[Java SE java.lang.String][1]

[1]: https://docs.oracle.com/javase/7/docs/api/java/lang/String.html
[2]: http://ol3q0aw97.bkt.clouddn.com/blog/java-string/java.lang.String.concat.png
