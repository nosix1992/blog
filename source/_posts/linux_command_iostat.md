---
title: Linux常用运维命令(iostat)笔记整理(一)
date: 2015-03-07 19:46:26
tags: 
- Command
categories:
- Linux
---

在linux服务器开发过程中， 经常需要各种命令配合来查看各种状态，所以整理了一些老的笔记来备忘。

# **iostat**

> iostat主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息

 - -c 仅显示CPU统计信息.与-d选项互斥.
 - -d 仅显示磁盘统计信息.与-c选项互斥.
 - -k 以K为单位显示每秒的磁盘请求数,默认单位块.
 - -t  在输出数据时,打印搜集数据的时间.
 - -V 打印版本号和帮助信息.
 - -x  输出扩展信息.
 

**. . .**<!-- more -->

------------
## *iostat常用用法1：iostat -d*
指定采样时间间隔与采样次数

我们可以以"iostat interval [count] ”形式指定iostat命令的采样间隔和采样次数：
```
linux # iostat -d 1 2
Linux 2.6.16.60-0.21-smp (linux)     06/13/12

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
sda               0.55         8.93        36.27    6737086   27367728
sdb               0.00         0.00         0.00        928          0

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
sda               2.00         0.00        72.00          0         72
sdb               0.00         0.00         0.00          0          0
```
以上命令输出Device的信息，采样时间为1秒，采样2次，若不指定采样次数，则iostat会一直输出采样信息，直到按”ctrl+c”退出命令。注意，第1次采样信息与单独执行iostat的效果一样，为从系统开机到当前执行时刻的统计信息。

----------
## *iostat常用用法2： iostat -xdk*
```
linux # iostat -xdk 1
Linux 2.6.16.60-0.21-smp (linux)     06/13/12

……
Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sda               0.00  9915.00    1.00   90.00     4.00 34360.00   755.25    11.79  120.57   6.33  57.60
```

以上各列的含义如下：

- rrqm/s: 每秒对该设备的读请求被合并次数，文件系统会对读取同块(block)的请求进行合并
- wrqm/s: 每秒对该设备的写请求被合并次数
- r/s: 每秒完成的读次数
- w/s: 每秒完成的写次数
- rkB/s: 每秒读数据量(kB为单位)
- wkB/s: 每秒写数据量(kB为单位)
- avgrq-sz:平均每次IO操作的数据量(扇区数为单位)
- avgqu-sz: 平均等待处理的IO请求队列长度
- await: 平均每次IO请求等待时间(包括等待时间和处理时间，毫秒为单位)
- svctm: 平均每次IO请求的处理时间(毫秒为单位)
- %util: 采用周期内用于IO操作的时间比率，即IO队列非空的时间比率
 

对于以上示例输出，我们可以获取到以下信息：

每秒向磁盘上写30M左右数据(wkB/s值)
每秒有91次IO操作(r/s+w/s)，其中以写操作为主体
平均每次IO请求等待处理的时间为120.57毫秒，处理耗时为6.33毫秒
等待处理的IO请求队列中，平均有11.79个请求驻留

