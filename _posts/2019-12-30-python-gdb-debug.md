---
layout: post
title:  "Python Debug with GDB"
date:   2020-12-30 15:04:02 +0800
---

为什么需要使用GDB来调试程序？

1. 线上问题排查，需要Attach到运行中的程序
2. 排查死锁问题

主要是用到了GDB可以附加到运行中的程序这个特性。



Mac 缺少相关的组件， 所以我使用 Ubuntu 环境来调试Python程序。


1.  安装 gdb 的 Python拓展

  ```
  apt-get install python3.7-dbg
  ```

  注意！设置权限:

  设置 /proc/sys/kernel/yama/ptrace_scope 内容为 0

  ```
  echo 0 |sudo tee /proc/sys/kernel/yama/ptrace_scope
  ```

  如果在docker 中调试，请在运行时加 --privileged 参数

  ```
  docker run --privileged -d -ti <image> bash
  docker exec --privileged -ti <container> bash
  ```

2. 运行 gdb， 有两种方式

  a)  使用gdb 启动程序

  ```
  # 交互式
  $ gdb python
  (gdb) run <programname>.py <arguments>
  
  # 自动式
  gdb -ex r --args python <programname>.py <arguments>
  ```

  b) 将 gdb attach 到正在运行的Python程序

  ```
  gdb python <pid of running process>
  ```

  attach 成功会暂停称在运行的程序，使用 c 命令继续

3. 调试程序

   Gdb 的命令非常丰富：可以加断点、打印栈、watch变量、单步等等。可以参考  [gdb调试命令](https://www.cnblogs.com/wuyuegb2312/archive/2013/03/29/2987025.html)

   这里主要介绍一下如何用 gdb 排查阻塞问题：阻塞的时候可以调用 `py-list` 或者 `py-bt` 查看当前代码阻塞的地方。一般是某个阻塞的系统调用，或者是等待获取某个锁。

   

参考文档：

1. [Debugging with GDB] (https://wiki.python.org/moin/DebuggingWithGdb)
2. [gdb调试命令](https://www.cnblogs.com/wuyuegb2312/archive/2013/03/29/2987025.html)



  
