# 13. Linux平台进程及作业管理
本章我们来学习 Linux 中的进程管理，这是运维的基本内容，我们需要借此查看 Linux 服务的负载，分析和删除系统上的异常进程等等。首先我们会简单介绍操作系统原理中有关进程，虚拟内存相关的基础知识，这是原理部分；如果对进程一无所知对很多命令的输出结果我们很难明白其含义。然后我们会介绍 Linux 上常用的进程查看和管理工具，这些都是我们分析和管理系统的重要工具，最后我们会说一说 Linux 中的作业管理。本章内容总结如下:
1. 操作系统的基本原理之进程
2. 常用的进程查看和管理工具
  - 进程查看命令: `pstree, ps, pidof, pgrep`
  - 进程管理命令: `kill, pkill, killall，nice, renice`
2. 实时动态命令的使用，`top, glances, htop`
2. 系统状态查看,`vmstat, pmap, dstat`
3. 作业管理

有关 Linux 的实现有两本书推荐大家观阅读
- [Linux内核设计与实现](https://book.douban.com/subject/6097773/): 入门级
- [深入理解Linux内核](https://book.douban.com/subject/2287506/): 进阶级
