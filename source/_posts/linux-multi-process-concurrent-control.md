---
title: Linux Shell多进程并发控制
date: 2016-01-10
tags:
- linux
- shell
categories:
- linux
toc: true
---
# 基础知识准备
## linux后台进程
Unix是一个多任务系统，允许多用户同时运行多个程序。shell的元字符`&`提供了在后台运行不需要键盘输入的程序的方法。输入命令后，其后紧跟`&`字符，该命令就会被送往到linux后台执行，而终端又可以继续输入下一个命令了。

比如：
```bash
sh a.sh &
sh b.sh &
sh c.sh &
```
这三个命令就会被**同时**送往linux后台执行，在这个程度上，认为这三个命令**并发执行**了。

<!-- more -->

## linux文件描述符
文件描述符（缩写fd）在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。每一个unix进程，都会拥有三个标准的文件描述符，来对应三种不同的流：

| 文件描述符 | 名称          |
| :------- | :-------------- |
| 0        | Standard Input  |
| 1        | Standard Output |
| 2        | Standard Error  |

每一个文件描述符会对应一个打开文件，同时，不同的文件描述符也可以对应同一个打开文件；同一个文件可以被不同的进程打开，也可以被同一个进程多次打开。

在`/proc/PID/fd`中，列举了进程`PID`所拥有的文件描述符，例如
```bash
#!/bin/bash
source /etc/profile;

# $$表示当前进程的PID
PID=$$

# 查看当前进程的文件描述符指向
ll /proc/$PID/fd
echo "-------------------";echo

# 文件描述符1与文件tempfd1进行绑定
( [ -e ./tempfd1 ] || touch ./tempfd1 ) && exec 1<>./tempfd1

# 查看当前进程的文件描述符指向
ll /proc/$PID/fd
echo "-------------------";echo;
```
```bash
[ouyangyewei@localhost learn_linux]$ sh learn_redirect.sh
total 0
lrwx------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 0 -> /dev/pts/0
lrwx------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 1 -> /dev/pts/0
lrwx------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 2 -> /dev/pts/0
lr-x------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 255 -> /home/ouyangyewei/workspace/learn_linux/learn_redirect.sh
-------------------

[ouyangyewei@localhost learn_linux]$ cat tempfd1
total 0
lrwx------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 0 -> /dev/pts/0
lrwx------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 1 -> /home/ouyangyewei/workspace/learn_linux/tempfd1
lrwx------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 2 -> /dev/pts/0
lr-x------. 1 ouyangyewei ouyangyewei 64 Jan  4 22:17 255 -> /home/ouyangyewei/workspace/learn_linux/learn_redirect.sh
-------------------
```
上述的例子中第12行，将文件描述符1与文件`tempfile`进行了绑定，此后，文件描述符1指向了`tempfile`文件，标准输出被重定向到了文件`tempfile`中。

## linux管道
在Unix或类Unix操作系统中，管道是一个由标准输入输出链接起来的进程集合，因此，每一个进程的输出将直接作为下一个进程的输入，

linux管道包含两种：

* 匿名管道
* 命名管道

***管道有一个特点，如果管道中没有数据，那么取管道数据的操作就会滞留，直到管道内进入数据，然后读出后才会终止这一操作；同理，写入管道的操作如果没有读取管道的操作，这一动作就会滞留。***

### 匿名管道
在Unix或类Unix操作系统的命令行中，匿名管道使用ASCII中垂直线`|`作为匿名管道符，匿名管道的两端是两个普通的，匿名的，打开的文件描述符：一个**只读端**和一个**只写端**，这就让其它进程无法连接到该匿名管道。

例如：
``` bash
cat file | less
```
为了执行上面的指令，Shell创建了两个进程来分别执行`cat`和`less`。下图展示了这两个进程是如何使用管道的：
![linux匿名管道][1]
有一点值得注意的是两个进程都连接到了管道上，这样写入进程`cat`就将其标准输出（文件描述符为`fd 1`）连接到了管道的写入端，读取进程`less`就将其标准输入（文件描述符为`fd 0`）连接到了管道的读入端。实际上，这两个进程并不知道管道的存在，它们只是从标准文件描述符中读取数据和写入数据。shell必须要完成相关的工作。

### 命名管道（FIFO，First In First Out）
命名管道也称FIFO，从语义上来讲，FIFO其实与匿名管道类似，但值得注意：

* 在文件系统中，FIFO拥有名称，并且是以设备特俗文件的形式存在的；
* 任何进程都可以通过FIFO共享数据；
* 除非FIFO两端同时有读与写的进程，否则FIFO的数据流通将会阻塞；
* 匿名管道是由shell自动创建的，存在于内核中；而FIFO则是由程序创建的（比如`mkfifo`命令），存在于文件系统中；
* 匿名管道是单向的字节流，而FIFO则是双向的字节流；

比如，可以利用FIFO实现单服务器、多客户端的应用程序:
![linux命名管道][2]

--------------

有了上面的知识准备，现在可以开始讲述，**linux多进程并发时，如何控制每次并发的进程数。**

# linux多进程并发数控制
最近小A需要生产2015年全年的KPI数据报表，现在小A已经将生产脚本写好了，生产脚本一次只能生产指定一天的KPI数据，假设跑一次生产脚本需要5分钟，那么：
* 如果是循环顺序执行，那么需要时间：5 * 365 = 1825 分钟，约等于 6 天
* 如果是一次性放到linux后台并发执行，365个后台任务，系统可承受不住哦！

既然不能一次性把365个任务放到linux后台执行，那么，能不能实现自动地每次将N个任务放到后台并发执行呢？当然是可以的啦。

``` bash
#! /bin/bash
source /etc/profile;

# -----------------------------

tempfifo=$$.fifo        # $$表示当前执行文件的PID
begin_date=$1           # 开始时间
end_date=$2             # 结束时间

if [ $# -eq 2 ]
then
    if [ "$begin_date" \> "$end_date" ]
    then
        echo "Error! $begin_date is greater than $end_date"
        exit 1;
    fi
else
    echo "Error! Not enough params."
    echo "Sample: sh loop_kpi 2015-12-01 2015-12-07"
    exit 2;
fi

# -----------------------------

trap "exec 1000>&-;exec 1000<&-;exit 0" 2
mkfifo $tempfifo
exec 1000<>$tempfifo
rm -rf $tempfifo

for ((i=1; i<=8; i++))
do
    echo >&1000
done

while [ $begin_date != $end_date ]
do
    read -u1000
    {
        echo $begin_date
        hive -f kpi_report.sql --hivevar date=$begin_date
        echo >&1000
    } &

    begin_date=`date -d "+1 day $begin_date" +"%Y-%m-%d"`
done

wait
echo "done!!!!!!!!!!"
```
* 第6～22行：比如：`sh loop_kpi_report.sh 2015-01-01 2015-12-01`：
  * `$1`表示脚本入参的第一个参数，等于2015-01-01
  * `$2`表示脚本入参的第二个参数，等于2015-12-01
  * `$#`表示脚本入参的个数，等于2  
  * 第13行用于比较传入的两个日期的大小，`\>`是转义
* 第26行：表示在脚本运行过程中，如果接收到`Ctrl+C`中断命令，则关闭文件描述符1000的读写，并正常退出
  * `exec 1000>&-;`表示关闭文件描述符1000的写
  * `exec 1000<&-;`表示关闭文件描述符1000的读
  * trap是捕获中断命令
* 第27～29行：
  * 第27行，创建一个管道文件
  * 第28行，将文件描述符1000与FIFO进行绑定，`<`读的绑定，`>`写的绑定，`<>`则标识对文件描述符1000的所有操作等同于对管道文件`$tempfifo`的操作
  * 第29行，可能会有这样的疑问：为什么不直接使用管道文件呢？事实上这并非多此一举，管道的一个重要特性，就是读写必须同时存在，缺失某一个操作，另一个操作就是滞留，而第28行的绑定文件描述符（读、写绑定）正好解决了这个问题
* 第31～34行：对文件描述符1000进行写入操作。通过循环写入8个空行，这个8就是我们要定义的后台并发的线程数。为什么是写空行而不是写其它字符？因为管道文件的读取，是以行为单位的
* 第37～42行：
  * 第37行，`read -u1000`的作用就是读取管道中的一行，在这里就是读取一个空行；每次读取管道就会减少一个空行
  * 第39～41行，注意到第42行结尾的`&`吗？它表示进程放到linux后台中执行
  * 第41行，执行完后台任务之后，往文件描述符1000中写入一个空行。这是关键所在了，由于`read -u1000`每次操作，都会导致管道减少一个空行，当linux后台放入了8个任务之后，由于文件描述符1000没有可读取的空行，将导致`read -u1000`一直处于等待。

-------------------

# 参考资料
* Unix Power Tools
* UNIX系统编程手册
* UNIX管道：[https://zh.wikipedia.org/wiki/%E7%AE%A1%E9%81%93_(Unix)][3]


[1]: http://ol3q0aw97.bkt.clouddn.com/blog/linux-multi-process-concurrent-control/linux-unnamed-pipe.png
[2]: http://ol3q0aw97.bkt.clouddn.com/blog/linux-multi-process-concurrent-control/linux-named-pipe.png
[3]: https://zh.wikipedia.org/wiki/%E7%AE%A1%E9%81%93_(Unix)
