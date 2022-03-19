---
title: linux关机命令
comments: false
tags: shutdown
categories: Linux
abbrlink: 2292
date: 2021-10-02 11:39:17
translate_title: Linux-shutdown-command
---

### 1. shutdown命令

shutdown会给系统计划一个时间关机。它可以被用于停止、关机、重启机器。
你可以指定一个时间字符串（通常是 now或者用hh:mm 指定小时/分钟）作为第一个参数。

shutdown命令示例：
```shell
shutdown

shutdown now  #立即关机

shutdown 13:20  # 下午13：20关机

shutdown -p now  ### 关闭机器

shutdown -H now  ### 停止机器      

shutdown -r09:35 ### 在 09:35am 重启机器

shutdown -c  ## 取消关机
```

### 2. halt 命令

halt通知硬件来停止所有的 CPU 功能，但是仍然保持通电。你可以用它使系统处于低层维护状态。
注意在有些情况会它会完全关闭系统。

halt 命令示例：
```shell
halt             ### 停止机器

halt -p          ### 关闭机器

halt --reboot    ### 重启机器
```

### 3.poweroff 命令
poweroff会发送一个 ACPI 信号来通知系统关机

```shell
poweroff           ### 关闭机器

poweroff --halt    ### 停止机器

poweroff --reboot  ### 重启机器
```

### 4.reboot 命令

reboot 通知系统重启。
```shell
reboot           ### 重启机器

reboot --halt    ### 停止机器

reboot -p        ### 关闭机器
```

### 5. init 命令
一. init是Linux系统操作中不可缺少的程序之一。
所谓的init进程，它是一个由内核启动的用户级进程。
内核自行启动（已经被载入内存，开始运行，并已初始化所有的设备驱动程序和数据结构等）之后，就通过启动一个用户级程序init的方式，完成引导进程。所以,init始终是第一个进程（其进程编号始终为1）。
内核会在过去曾使用过init的几个地方查找它，它的正确位置（对Linux系统来说）是/sbin/init。如果内核找不到init，它就会试着运行/bin/sh，如果运行失败，系统的启动也会失败。

二. init一共分为7个级别，这7个级别的所代表的含义如下

0：停机或者关机（千万不能将initdefault设置为0）

1：单用户模式，只root用户进行维护

2：多用户模式，不能使用NFS(Net File System)

3：完全多用户模式（标准的运行级别）

4：安全模式

5：图形化（即图形界面）

6：重启（千万不要把initdefault设置为6）
