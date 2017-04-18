---
title: Java Class文件浅析
categories:
- java
tags:
- java
date: 2017-04-03
toc: true
---

> 此文仅作为学习笔记，本篇文章只记录Java的Class文件的部分信息

``` java
class TBase {
    public void f0() {}
}

public class Turtle extends TBase {
    private int t1;
    private static int t2;

    // 常量，放入常量池中
    public static final int t3 = 65535;

    public void f1() {
        int i = 1;
    }
    public void f2(int arg2) {
        int newArg = arg2;
    }
    public void f3(long arg3) {
        long newArg = arg3;
    }
    public static void main(String[] args) {
        // 编译期优化，
        // 直接将"a","b","c"作为字符串常量，
        // 放入常量池中
        String a = "a";
        String b = "b";
        String c = "c";
        String d = a + b + c;
    }
}
```

1. 编译生成详细信息
```
javac -g Turtle.java      编译，生成详细信息
javap -v Turtle.class     反编译，查看详细信息
```

2. linux下查看二进制文件
```
hexdump -C Turtle.class   显示十六进制码和ASCII码对照表
```

---

### 常量池
javap命令计算出来的常量池，参见[class文件中的常量池](#class文件中的常量池)，其中重点关注这几个常量：
```
Constant pool:
  #2 = String             #39            // a
  #3 = String             #41            // b
  #4 = String             #42            // c
  #17 = Integer           65535
  #39 = Utf8              a
  #41 = Utf8              b
  #42 = Utf8              c
```

1. 使用final声明的常量，将放入到常量池中
``` java
public static final int t3 = 65535; // 常量，放入常量池中
```
2. 在编译期间优化，直接将 ***a***、***b***、***c*** 作为字符串常量，放入到常量池中
``` java
String a = "a";   // 编译期优化，直接将a作为字符串常量，放入常量池中
String b = "b";
String c = "c";
```

---

### 类索引、父类索引与接口索引集合
![img_001][1]
从偏移地址0x00000206开始的3个值分别是0x000A，0x000B，0x0000，也就是：
类索引=10
父类索引=11，
接口索引集合大小=0
对应javap命令计算出来的常量池，找到对应的类和父类的常量，如下：
```
#10 = Class              #52            // Turtle
#11 = Class              #53            // TBase
#52 = Utf8               Turtle
#53 = Utf8               TBase
```

---

### 字段表集合
![img_002][2]
从偏移地址0x00000206开始，第一个值是字段数=0x0003，即有三个成员变量，然后依次：
t1: `00 02 00 0C 00 0D 00 00`
t2: `00 0a 00 0e 00 0d 00 00`
t3: `00 19 00 0f 00 0d 00 01 00 10 00 00 00 02 00 11`

以t2举例解释
![img_003][3]

---

### 方法表集合
![img_004][4]
从偏移地址0x0000022E开始，第一个值是字段数=0x0006，即有6个方法，参见字节码可见具体的6个方法：
``` java
1.8.0_111
Compiled from "Turtle.java"
public class Turtle extends TBase {
  public static final int t3;
  public Turtle();
  public void f1();
  public void f2(int);
  public void f3(long);
  public static void main(java.lang.String[]);
  public void f0();
}
```

以f2方法举例解释
![img_005][5]
访问标记=0x0001，表示***ACC_PUBLIC***，方法是***public***的；
方法名称索引=0x001b=27，对应常量表中的***f2***；
描述符索引=0x001c=28，对应常量表中的***(I)V***；
属性计数器=0x0001=1，表示此方法的属性表集合有一项属性；
属性名称索引=0x0014=20，对应常量表中的***Code***，说明此属性是方法的字节码描述;
(关于Code属性，建议看 **《深入理解Java虚拟机（第2版）》的"§ 6.3.7 属性表集合"** )

---

### 附录
#### class文件中的常量池
``` java
Constant pool:
   #1 = Methodref          #11.#47        // TBase."<init>":()V
   #2 = String             #39            // a
   #3 = String             #41            // b
   #4 = String             #42            // c
   #5 = Class              #48            // java/lang/StringBuilder
   #6 = Methodref          #5.#47         // java/lang/StringBuilder."<init>":()V
   #7 = Methodref          #5.#49         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #5.#50         // java/lang/StringBuilder.toString:()Ljava/lang/String;
   #9 = Methodref          #11.#51        // TBase.f0:()V
  #10 = Class              #52            // Turtle
  #11 = Class              #53            // TBase
  #12 = Utf8               t1
  #13 = Utf8               I
  #14 = Utf8               t2
  #15 = Utf8               t3
  #16 = Utf8               ConstantValue
  #17 = Integer            65535
  #18 = Utf8               <init>
  #19 = Utf8               ()V
  #20 = Utf8               Code
  #21 = Utf8               LineNumberTable
  #22 = Utf8               LocalVariableTable
  #23 = Utf8               this
  #24 = Utf8               LTurtle;
  #25 = Utf8               f1
  #26 = Utf8               i
  #27 = Utf8               f2
  #28 = Utf8               (I)V
  #29 = Utf8               arg2
  #30 = Utf8               newArg
  #31 = Utf8               f3
  #32 = Utf8               (J)V
  #33 = Utf8               arg3
  #34 = Utf8               J
  #35 = Utf8               main
  #36 = Utf8               ([Ljava/lang/String;)V
  #37 = Utf8               args
  #38 = Utf8               [Ljava/lang/String;
  #39 = Utf8               a
  #40 = Utf8               Ljava/lang/String;
  #41 = Utf8               b
  #42 = Utf8               c
  #43 = Utf8               d
  #44 = Utf8               f0
  #45 = Utf8               SourceFile
  #46 = Utf8               Turtle.java
  #47 = NameAndType        #18:#19        // "<init>":()V
  #48 = Utf8               java/lang/StringBuilder
  #49 = NameAndType        #54:#55        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #50 = NameAndType        #56:#57        // toString:()Ljava/lang/String;
  #51 = NameAndType        #44:#19        // f0:()V
  #52 = Utf8               Turtle
  #53 = Utf8               TBase
  #54 = Utf8               append
  #55 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #56 = Utf8               toString
  #57 = Utf8               ()Ljava/lang/String;
```

#### class文件的十六进制码和ASCII码
```
00000000  ca fe ba be 00 00 00 34  00 3a 0a 00 0b 00 2f 08  |.......4.:..../.|
00000010  00 27 08 00 29 08 00 2a  07 00 30 0a 00 05 00 2f  |.'..)..*..0..../|
00000020  0a 00 05 00 31 0a 00 05  00 32 0a 00 0b 00 33 07  |....1....2....3.|
00000030  00 34 07 00 35 01 00 02  74 31 01 00 01 49 01 00  |.4..5...t1...I..|
00000040  02 74 32 01 00 02 74 33  01 00 0d 43 6f 6e 73 74  |.t2...t3...Const|
00000050  61 6e 74 56 61 6c 75 65  03 00 00 ff ff 01 00 06  |antValue........|
00000060  3c 69 6e 69 74 3e 01 00  03 28 29 56 01 00 04 43  |<init>...()V...C|
00000070  6f 64 65 01 00 0f 4c 69  6e 65 4e 75 6d 62 65 72  |ode...LineNumber|
00000080  54 61 62 6c 65 01 00 12  4c 6f 63 61 6c 56 61 72  |Table...LocalVar|
00000090  69 61 62 6c 65 54 61 62  6c 65 01 00 04 74 68 69  |iableTable...thi|
000000a0  73 01 00 08 4c 54 75 72  74 6c 65 3b 01 00 02 66  |s...LTurtle;...f|
000000b0  31 01 00 01 69 01 00 02  66 32 01 00 04 28 49 29  |1...i...f2...(I)|
000000c0  56 01 00 04 61 72 67 32  01 00 06 6e 65 77 41 72  |V...arg2...newAr|
000000d0  67 01 00 02 66 33 01 00  04 28 4a 29 56 01 00 04  |g...f3...(J)V...|
000000e0  61 72 67 33 01 00 01 4a  01 00 04 6d 61 69 6e 01  |arg3...J...main.|
000000f0  00 16 28 5b 4c 6a 61 76  61 2f 6c 61 6e 67 2f 53  |..([Ljava/lang/S|
00000100  74 72 69 6e 67 3b 29 56  01 00 04 61 72 67 73 01  |tring;)V...args.|
00000110  00 13 5b 4c 6a 61 76 61  2f 6c 61 6e 67 2f 53 74  |..[Ljava/lang/St|
00000120  72 69 6e 67 3b 01 00 01  61 01 00 12 4c 6a 61 76  |ring;...a...Ljav|
00000130  61 2f 6c 61 6e 67 2f 53  74 72 69 6e 67 3b 01 00  |a/lang/String;..|
00000140  01 62 01 00 01 63 01 00  01 64 01 00 02 66 30 01  |.b...c...d...f0.|
00000150  00 0a 53 6f 75 72 63 65  46 69 6c 65 01 00 0b 54  |..SourceFile...T|
00000160  75 72 74 6c 65 2e 6a 61  76 61 0c 00 12 00 13 01  |urtle.java......|
00000170  00 17 6a 61 76 61 2f 6c  61 6e 67 2f 53 74 72 69  |..java/lang/Stri|
00000180  6e 67 42 75 69 6c 64 65  72 0c 00 36 00 37 0c 00  |ngBuilder..6.7..|
00000190  38 00 39 0c 00 2c 00 13  01 00 06 54 75 72 74 6c  |8.9..,.....Turtl|
000001a0  65 01 00 05 54 42 61 73  65 01 00 06 61 70 70 65  |e...TBase...appe|
000001b0  6e 64 01 00 2d 28 4c 6a  61 76 61 2f 6c 61 6e 67  |nd..-(Ljava/lang|
000001c0  2f 53 74 72 69 6e 67 3b  29 4c 6a 61 76 61 2f 6c  |/String;)Ljava/l|
000001d0  61 6e 67 2f 53 74 72 69  6e 67 42 75 69 6c 64 65  |ang/StringBuilde|
000001e0  72 3b 01 00 08 74 6f 53  74 72 69 6e 67 01 00 14  |r;...toString...|
000001f0  28 29 4c 6a 61 76 61 2f  6c 61 6e 67 2f 53 74 72  |()Ljava/lang/Str|
00000200  69 6e 67 3b 00 21 00 0a  00 0b 00 00 00 03 00 02  |ing;.!..........|
00000210  00 0c 00 0d 00 00 00 0a  00 0e 00 0d 00 00 00 19  |................|
00000220  00 0f 00 0d 00 01 00 10  00 00 00 02 00 11 00 06  |................|
00000230  00 01 00 12 00 13 00 01  00 14 00 00 00 2f 00 01  |............./..|
00000240  00 01 00 00 00 05 2a b7  00 01 b1 00 00 00 02 00  |......*.........|
00000250  15 00 00 00 06 00 01 00  00 00 09 00 16 00 00 00  |................|
00000260  0c 00 01 00 00 00 05 00  17 00 18 00 00 00 01 00  |................|
00000270  19 00 13 00 01 00 14 00  00 00 3b 00 01 00 02 00  |..........;.....|
00000280  00 00 03 04 3c b1 00 00  00 02 00 15 00 00 00 0a  |....<...........|
00000290  00 02 00 00 00 0f 00 02  00 10 00 16 00 00 00 16  |................|
000002a0  00 02 00 00 00 03 00 17  00 18 00 00 00 02 00 01  |................|
000002b0  00 1a 00 0d 00 01 00 01  00 1b 00 1c 00 01 00 14  |................|
000002c0  00 00 00 45 00 01 00 03  00 00 00 03 1b 3d b1 00  |...E.........=..|
000002d0  00 00 02 00 15 00 00 00  0a 00 02 00 00 00 13 00  |................|
000002e0  02 00 14 00 16 00 00 00  20 00 03 00 00 00 03 00  |........ .......|
000002f0  17 00 18 00 00 00 00 00  03 00 1d 00 0d 00 01 00  |................|
00000300  02 00 01 00 1e 00 0d 00  02 00 01 00 1f 00 20 00  |.............. .|
00000310  01 00 14 00 00 00 45 00  02 00 05 00 00 00 03 1f  |......E.........|
00000320  42 b1 00 00 00 02 00 15  00 00 00 0a 00 02 00 00  |B...............|
00000330  00 17 00 02 00 18 00 16  00 00 00 20 00 03 00 00  |........... ....|
00000340  00 03 00 17 00 18 00 00  00 00 00 03 00 21 00 22  |.............!."|
00000350  00 01 00 02 00 01 00 1e  00 22 00 03 00 09 00 23  |.........".....#|
00000360  00 24 00 01 00 14 00 00  00 84 00 02 00 05 00 00  |.$..............|
00000370  00 22 12 02 4c 12 03 4d  12 04 4e bb 00 05 59 b7  |."..L..M..N...Y.|
00000380  00 06 2b b6 00 07 2c b6  00 07 2d b6 00 07 b6 00  |..+...,...-.....|
00000390  08 3a 04 b1 00 00 00 02  00 15 00 00 00 16 00 05  |.:..............|
000003a0  00 00 00 1b 00 03 00 1c  00 06 00 1d 00 09 00 1e  |................|
000003b0  00 21 00 1f 00 16 00 00  00 34 00 05 00 00 00 22  |.!.......4....."|
000003c0  00 25 00 26 00 00 00 03  00 1f 00 27 00 28 00 01  |.%.&.......'.(..|
000003d0  00 06 00 1c 00 29 00 28  00 02 00 09 00 19 00 2a  |.....).(.......*|
000003e0  00 28 00 03 00 21 00 01  00 2b 00 28 00 04 10 41  |.(...!...+.(...A|
000003f0  00 2c 00 13 00 01 00 14  00 00 00 2f 00 01 00 01  |.,........./....|
00000400  00 00 00 05 2a b7 00 09  b1 00 00 00 02 00 15 00  |....*...........|
00000410  00 00 06 00 01 00 00 00  09 00 16 00 00 00 0c 00  |................|
00000420  01 00 00 00 05 00 17 00  18 00 00 00 01 00 2d 00  |..............-.|
00000430  00 00 02 00 2e                                    |.....|
00000435
```


[1]: http://ol3q0aw97.bkt.clouddn.com/blog/java-classfile/img_001.png
[2]: http://ol3q0aw97.bkt.clouddn.com/blog/java-classfile/img_002.png
[3]: http://ol3q0aw97.bkt.clouddn.com/blog/java-classfile/img_003.png
[4]: http://ol3q0aw97.bkt.clouddn.com/blog/java-classfile/img_004.png
[5]: http://ol3q0aw97.bkt.clouddn.com/blog/java-classfile/img_005.png
