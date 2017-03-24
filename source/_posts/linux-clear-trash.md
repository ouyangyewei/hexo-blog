---
title: linux与hdfs历史文件清理脚本
categories:
- linux
tags:
- linux
- shell
date: 2016-04-18
toc: true
---

# 功能介绍
* 清理指定天数前的linux文件（支持指定清理文件夹、豁免清理文件夹）
* 清理指定天数前的hdfs文件

# 用法
### 加入killtask命令
1. 下载 [killtask][1] 命令到本地服务器
2. 编辑`/etc/profile`或`~/.bashrc`文件，加入如下代码行
```bash
alias killtask='sh your_linux_path/killtask'
```
3. 使用命令`source /etc/profile`或`source ~/.bashrc`，使脚本立即生效

### crontab设置clear_trash脚本
1. 下载 [clear_trash][2] 脚本到本地服务器
2. 加入crontab配置
``` bash
* */6 * * * (sh /your_linux_path/killtask clear_trash > /dev/null; sh /your_linux_path/clear_trash.sh >> /your_linux_path/xxx.log 2>&1 &)
```

[1]: https://github.com/ouyangyewei/Toolkit/tree/master/linux/killtask
[2]: https://github.com/ouyangyewei/Toolkit/blob/master/linux/clear_trash/clear_trash.sh
