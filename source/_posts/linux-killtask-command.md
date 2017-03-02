---
title: killtask命令
categories:
- linux
tags:
- linux
- shell
date: 2016-04-27
toc: true
---

# 功能介绍
方便杀死指定名称的linux进程
<!-- more -->

# 用法
1. 下载`killtask`命令到本地服务器
2. 编辑`/etc/profile`或`~/.bashrc`文件，加入如下代码行
```bash
alias killtask='sh your_linux_path/killtask'
```
3. 使用命令`source /etc/profile`或`source ~/.bashrc`，使脚本立即生效

# 源码
https://github.com/ouyangyewei/Toolkit/tree/master/linux/killtask

# 样例
![killtask_demo][1]

[1]: http://ol3q0aw97.bkt.clouddn.com/blog/linux-killtask-command/killtask_sample.gif
