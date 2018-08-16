# 20.3 apache httpd 简介
httpd 是 ASF(apache software foundation, apache 软件基金会)下的顶级项目，也是目前市场占有率最高的 web 服务器。本节我们就来对 httpd 做一个概括行的介绍。在之后的章节我们再来详细的学习 httpd 的配置。

## 1. httpd 简介
### 1.1 httpd 版本
httpd 目前有如下主流的版本
- httpd 1.3：官方已经停止维护；
- httpd 2.0：
- httpd 2.2: Centos6 base 仓库的默认安装版本
- httpd 2.4：目前最新稳定版，Centos7 base 仓库的默认安装版本

httpd2.2 和 httpd2.4 目前都有使用，他们之间存在比较大的差异。在介绍 httpd 的配置时，我们将主要以 httpd2.4 为主，对于 http2.2 中差异较大的，我们会单独指出。

### 1.2 httpd的特性
httpd 具有如下的一些关键特性:
1. 高度模块化： core + modules
2. DSO：dynamic shared object 动态共享对象
3. MPM：Multipath processing Modules (多路处理模块)
    - `prefork`：多进程模型, 每个进程响应一个请求；
        - 一个主进程：负责生成子进程及回收子进程；负责创建套接字；负责接收请求，并将其派发给某子进程进行处理；
        - n个子进程：每个子进程处理一个请求；
        - 工作模型：会预先生成几个空闲进程，随时等待用于响应用户请求；有最大空闲数和最小空闲数；
    - `worker`：多进程多线程模型，每线程处理一个用户请求；
        - 一个主进程：负责生成子进程；负责创建套接字；负责接收请求，并将其派发给某子进程进行处理；
        - 多个子进程：每个子进程负责生成多个线程；
        - 每个线程：负责响应用户请求；
        - 并发响应数量：m*n
            - m：子进程数量
            - n：每个子进程所能创建的最大线程数量；
    - `event`：事件驱动模型，多进程模型，每个进程响应多个请求；
        - 一个主进程 ：负责生成子进程；负责创建套接字；负责接收请求，并将其派发给某子进程进行处理；
        - 子进程：基于事件驱动机制直接响应多个请求；
        - 适用版本:
            - httpd-2.2: 仍为测试使用模型；
            - httpd-2.4：event可生产环境中使用；


## 2. httpd安装
httpd 可以通过 base 仓库的 rpm 包直接安装，也可以编译安装。通常除非需要定制新功能，或其它原因，不建议采用编译安装的方式。一是对规模部署不便，二是安装过程繁琐，需要准备编译环境，还需要额外的配置。

### 2.1 httpd2.2 与 httpd2.4 的对比
下面是 httpd2.2 httpd2.4 安装，管理，以及配置文件路经的对比

|配置|httpd2.2|httpd2.4|
|:---|:---|:---|
|配置文件|`/etc/httpd/conf/httpd.conf`<br>`/etc/httpd/conf.d/*.conf`|`/etc/httpd/conf/httpd.conf`<br>`/etc/httpd/conf.d/*.conf`<br>`/etc/httpd/conf.modules.d/*.conf`(模块配置)|
|服务脚本|`/etc/rc.d/init.d/httpd`<br>`/etc/sysconfig/httpd`(脚本配置)|`/usr/lib/systemd/system/httpd.service`|
|主程序|`/usr/sbin/httpd`<br>`/usr/sbin/httpd.event`<br>`/usr/sbin/httpd.worker`|`/usr/sbin/httpd`<br>支持MPM的动态切换|
|访问日志|`/var/log/httpd/access_log`|`/var/log/httpd/access_log`|
|错误日志|`/var/log/httpd/error_log`|`/var/log/httpd/error_log`|
|站点文档|`/var/www/html`|`/var/www/html`|
|模块文件|`/usr/lib64/httpd/modules`|`/usr/lib64/httpd/modules`|
|服务控制|`chkconfig  httpd  on,off`|`systemctl  enable,disable  httpd.service`|
||`service  {start,stop,restart,status,reload}  httpd`|`systemctl  {start,stop,restart,status}  httpd`|

### 2.2 http2.4 rpm 包
```
$  yum list all httpd*
httpd.x86_64                       2.4.6-80.el7.centos.1                updates
httpd-devel.x86_64                 2.4.6-80.el7.centos.1                updates
httpd-itk.x86_64                   2.4.7.04-2.el7                       epel    
httpd-manual.noarch                2.4.6-80.el7.centos.1                updates
httpd-tools.x86_64                 2.4.6-80.el7.centos.1                updates
```
httpd 相关rmp 包:
1. `httpd`: httpd web 服务的主程序包
2. `httpd-tools`: httpd 相关的辅助工具
3. `httpd-manual`: httpd 文档

#### httpd
```
$ rpm -ql httpd|grep -v share
/etc/httpd                       # 配置文件
/etc/httpd/conf   
/etc/httpd/conf.d
/etc/httpd/conf.d/*
/etc/httpd/conf.modules.d
/etc/httpd/conf/httpd.conf

/etc/httpd/modules              # httpd 模块所在目录
/usr/lib64/httpd/modules

/usr/sbin/apachectl             # httpd 相关的程序
/usr/sbin/fcgistarter
/usr/sbin/htcacheclean
/usr/sbin/httpd
/usr/sbin/rotatelogs
/usr/sbin/suexec
/var/cache/httpd
/var/cache/httpd/proxy

/var/cache/httpd               # 目录与日志文件
/var/cache/httpd/proxy
/var/lib/dav
/var/log/httpd
/var/www
/var/www/cgi-bin
/var/www/html
```

#### httpd-tools
```
$ rpm -ql httpd-tools
/usr/bin/ab           # web 服务测试工具
/usr/bin/htdbm
/usr/bin/htdigest
/usr/bin/htpasswd     # 
/usr/bin/httxt2dbm
/usr/bin/logresolve
```
