# 25. Linux日志管理、时钟服务、sudo
Linux 中有一些通用原则，比如如果上下层衔接不通畅，通常的解决方法就是添加一个中间层，虚拟文件系统就是最典型的一例，再比如如果一个功能被很多应用所需要，通常会做成一个公共服务以便，认证功能 pam 还有本节将要介绍的 日志服务 rsyslog 就是典型的示例。本章我们会讲一些类似这些的 Linux 中基础的但是并不复杂的基础服务，包括:
1. 时间同步服务 chrony
2. 日志管理服务 rsyslog
3. sudo 权限管理
