# 26.3 nginx main 配置段
nginx 配置文件有众多参数，因此我们按照配置文件的配置段分别讲解 nginx 配置。本节主要是 ngnix 核心配置段。这些参数可分为如下几个类别:
1. 正常运行的必备配置
2. 优化性能相关的配置
3. 事件相关的配置
4. 用于调试、定位问题

nginx 参数的详细配置可参阅 [dirindex](http://nginx.org/en/docs/dirindex.html)

## 1. 基础核心配置
最长需要修改的参数:
- `worker_process`
- `worker_connections`
- `worker_cpu_affinity`
- `worker_priority`

### 1.1 正常运行的必备配置：
```
user username [groupname];
# 指定运行worker进程的用户和组

pid /path/to/pidfile_name;
# 指定nginx的pid文件

worker_rlimit_nofile n;
# 指定所有worker进程所能够打开的最大文件句柄数；

worker_rlimit_sigpending n;
# 设定每个用户能够发往worker进程的信号的数量；
```

### 1.2 优化性能相关的配置：
```

worker_processes n;
# worker进程的个数；通常其数值应该为CPU的物理核心数减1；
worker_processes auto; # nginx 将根据 cpu 数量自动选择 worker 进程数


worker_cpu_affinity cpumask ...;
# 作用: 提升缓存命中率
# eg:
#    worker_processes 6;
#    worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000;
worker_cpu_affinity  auto;  # nginx 将根据 cpu 数量自动配置


ssl_engine device;  
# 在存在ssl硬件加速器的服务器上，指定所使用的ssl硬件加速设备；

timer_resolution t
# 作用:
#    每次内核事件调用返回时，都会使用gettimeofday()来更新nginx缓存时钟；
#    timer_resolution用于定义每隔多久才会由gettimeofday()更新一次缓存时钟；
#    x86-64系统上，gettimeofday()代价已经很小，可以忽略此配置；

worker_priority nice;
# 作用: 指明 work 进程的 nice 值,  -20,19之间的值
```

### 1.3 事件相关的配置
```
accept_mutex [on|off]
# 作用: 是否打开Ningx的负载均衡锁；
# 功能:
#      此锁能够让多个worker进轮流地、序列化地与新的客户端建立连接；
#      而通常当一个worker进程的负载达到其上限的7/8，master就尽可能不再将请求调度此worker；


accept_mutex_delay ms;
# 作用:
#    accept锁模式中，一个worker进程为取得accept锁的等待时长
#    如果某worker进程在某次试图取得锁时失败了，至少要等待ms才能再一次请求锁；

lock_file /path/to/lock_file;
# 作用: accept_mutex 用到的lock文件路径


use [epoll|rtsig|select|poll]
# 作用: 指明使用的IO模型，建议让Nginx 自动选择

work_connections  nums
# 设定单个 worker 进程能处理的最大并发连接数量, 尽可能大，以避免成为限制，eg: 51200

multi_accept on|off;
# 作用: 是否允许一次性地响应多个用户请求；默认为Off;
```

### 1.4 用于调试、定位问题: 只调试nginx时使用
```
daemon on|off;
# 作用:
#      是否以守护进程让ningx运行后台；默认为on，
#      调试时可以设置为off，使得所有信息去接输出控制台；

master_process on|off
# 作用:
#    是否以master/worker模式运行nginx；默认为on；
#    调试时可设置off以方便追踪；

error_log path level;
# 作用:
#    错误日志文件及其级别；默认为error级别；调试时可以使用debug级别，
#    但要求在编译时必须使用--with-debug启用debug功能；
# path: stderr | syslog:server=address[,paramter=value] | memory:size
# level: debug | info | notice|  warn|crit| alter| emreg
```
