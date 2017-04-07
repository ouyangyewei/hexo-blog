---
title: Java StringBuffer和StringBuilder
categories:
- java
tags:
- java
date: 2017-04-06
toc: true
---

### 简介
StringBuffer和StringBuilder，两者都是可变对象，都继承java.lang.AbstractStringBuilder类，都实现java.io.Serializable和java.lang.CharSequence接口。
最大的区别在于：<b><u>StringBuffer是线程安全的，而StringBuilder是非线程安全的</u></b>

下面代码摘自java.lang.StringBuffer
``` java
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
```

下面代码摘自java.lang.StringBuilder
``` java
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
```

![java_StringBuffer_StringBuilder][1]

---

### AbstractStringBuilder类
AbstractStringBuilder类封装了StringBuffer和StringBuilder大部分操作的实现。
#### 字符串的内存形态
下面代码摘自java.lang.AbstractStringBuilder
``` java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;
```
StringBuffer和StringBuilder没有具体的成员变量来存储字符串，而是使用继承自AbstractStringBuilder类的成员变量`char[] value`，因为没有使用final关键字修饰，因此值是可变的。

#### 字符串构造方法
下面代码摘自java.lang.StringBuffer
``` java
public StringBuffer() {
    super(16);
}
```
下面代码摘自java.lang.StringBuilder
``` java
public StringBuilder() {
    super(16);
}
```
下面代码摘自java.lang.AbstractStringBuilder
``` java
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```
当创建一个StirngBuffer或StringBuilder对象时，若不指定容量，则默认创建长度为16的char类型数组

#### 字符串的append操作
下面代码摘自java.lang.AbstractStringBuilder，以入参为String对象为例
``` java
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    // 检查是否char[]数组是否需要扩容
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}

private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    // value.length默认长度是16
    // minimumCapacity = str.length + 字符串的实际长度
    // 若当前字符串数组长度不足最小应分配的长度，则将重新创建一个长度的char[]数组
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}
```
![StringBuffer.append()][5]

#### 字符串的insert操作
下面代码摘自java.lang.AbstractStringBuilder，以入参为String对象为例
``` java
public AbstractStringBuilder insert(int offset, String str) {
    if ((offset < 0) || (offset > length()))
        throw new StringIndexOutOfBoundsException(offset);
    if (str == null)
        str = "null";
    int len = str.length();
    ensureCapacityInternal(count + len);
    System.arraycopy(value, offset, value, offset + len, count - offset);
    str.getChars(value, offset);
    count += len;
    return this;
}
```
![StringBuffer.insert][7]
假设执行如下代码：
``` java
StringBuffer sb = new StringBuffer("abghij");
sb.insert(2, "cdef");
```
![StringBuffer.insert.visio][8]

#### 字符串的delete操作
下面代码摘自java.lang.AbstractStringBuilder
``` java
public AbstractStringBuilder delete(int start, int end) {
    if (start < 0)
        throw new StringIndexOutOfBoundsException(start);
    if (end > count)
        end = count;
    if (start > end)
        throw new StringIndexOutOfBoundsException();
    int len = end - start;
    if (len > 0) {
        System.arraycopy(value, start+len, value, start, count-end);
        count -= len;
    }
    return this;
}
```
实际上的操作是字符串数组拷贝，假设执行如下代码：
``` java
StringBuffer sb = new StringBuffer("abcdefghij");
sb.delete(2, 6);
```
![StringBuffer.delete][6]

---

### StringBuffer类
#### 为什么是线程安全的
线程安全是指多线程操作同一个对象，不会出现同步等问题。StringBuffer类中，使用了大量的synchronized关键字来修饰方法。
摘取java.lang.StringBuffer部分使用synchronized关键字修饰的代码
``` java
@Override
public synchronized int length() {
    return count;
}

@Override
public synchronized int capacity() {
    return value.length;
}

@Override
public synchronized void ensureCapacity(int minimumCapacity) {
    super.ensureCapacity(minimumCapacity);
}
```

#### transient关键字
摘自[Java Language Specification, Java SE 7 Edition, Section 8.3.1.3. transient Fields][4]
> Variables may be marked transient to indicate that they are not part of the persistent state of an object.

在Java中，transient关键字用来指出哪些成员变量不应该被序列化。值得注意的是：
* 序列化针对的是对象，而不是类；
* static修饰的变量，本身是隐式的transient，同时静态变量是属于类层次，不能被序列化；
* transient只能用于修饰成员变量，不能修饰本地变量，不能修饰方法和类。

StringBuffer类中，有一个成员变量
``` java
/**
 * A cache of the last value returned by toString. Cleared
 * whenever the StringBuffer is modified.
 */
private transient char[] toStringCache;
```
toStringCache这个成员变量，从命名上看，猜测是为了用于toString()方法而做的字符串缓冲。可见，如果是为了做缓冲，确实没必要在StringBuffer对象中持久化。

#### toString的操作
下面代码摘自java.lang.StringBuffer
``` java
@Override
public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}
```
toStringCache获得实际长度的字符串数组，并创建一个String对象

---

### 参考材料
[Java SE java.lang.StringBuffer][2]
[Java SE java.lang.StringBuilder][3]
[Java transient关键字][4]

[1]: http://ol3q0aw97.bkt.clouddn.com/blog/java-stringbuffer-stringbuilder/StringBuffer_StringBuilder_UML.png
[2]: https://docs.oracle.com/javase/7/docs/api/java/lang/StringBuffer.html
[3]: https://docs.oracle.com/javase/7/docs/api/java/lang/StringBuilder.html
[4]: https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.3.1.3
[5]: http://ol3q0aw97.bkt.clouddn.com/blog/java-stringbuffer-stringbuilder/StringBuffer.append.png
[6]: http://ol3q0aw97.bkt.clouddn.com/blog/java-stringbuffer-stringbuilder/StringBuffer.delete.png
[7]: http://ol3q0aw97.bkt.clouddn.com/blog/java-stringbuffer-stringbuilder/StringBuffer.insert.png
[8]: http://ol3q0aw97.bkt.clouddn.com/blog/java-stringbuffer-stringbuilder/StringBuffer.insert_visio.png
