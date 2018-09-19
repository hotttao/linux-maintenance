# 32.1 haproxy 入门
在学习配置 haproxy 之前我们先来对其做个简单了解，看看其程序与配置文件结构。

## 1. haproxy 简介
### 1.1 主站与文档
对于 haproxy 介绍和性能优势这里就不多介绍，推荐大家多看看 haproxy 的主页和官方文档，特别是官方文档
1. 主站:
  - http://www.haproxy.org
  - http://www.haproxy.com
2. 文档: http://cbonte.github.io/haproxy-dconv/

### 1.2 程序结构
haproxy 已经被收录至 yum 的 base 仓库，可直接安装，下面是 rpm 包的程序结构
```
$ rpm -ql haproxy|egrep -v "(doc|man)"
/etc/haproxy
/etc/haproxy/haproxy.cfg               # 主配置文件
/usr/sbin/haproxy                      # 主程序
/usr/sbin/haproxy-systemd-wrapper
/usr/bin/halog                         # 辅助工具
/usr/bin/iprange
/usr/lib/systemd/system/haproxy.service # Unit file
/etc/sysconfig/haproxy                  # Unit file 的配置文件  
/usr/share/haproxy                      # 错误页
/usr/share/haproxy/400.http
/usr/share/haproxy/403.http
/usr/share/haproxy/408.http
/usr/share/haproxy/500.http
/usr/share/haproxy/502.http
/usr/share/haproxy/503.http
/usr/share/haproxy/504.http
/usr/share/haproxy/README
/var/lib/haproxy
/etc/logrotate.d/haproxy
```

### 1.3 配置文件结构
```
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  main *:80
    default_backend             app

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server      py 127.0.0.1:8888 check
```

haproxy 的配置分成两大配置段
1. `global`：全局配置段，用于定义
  - 进程及安全配置相关的参数
  - 性能调整相关参数
  - Debug参数
2. proxies：代理配置段，包括四小配置段
  - `defaults`：为frontend, listen, backend提供默认配置；
  - `fronted`：定义前端监听的服务，相当于nginx, `server {}`
  - `backend`：定义后端服务器组，相当于nginx, `upstream {}`
  - `listen`：后端服务器组与前端服务一一对应时的便捷配置方式，可同时定义前端与后端


### 1.4 haproxy 简单配置示例
下面是两个配置示例，我们会在下一节详细介绍 haproxy 各个重要配置选项。
```
frontend web
  bind *:80
  default_backend     websrvs

backend websrvs
  balance roundrobin                 # 定义调度算法
  server srv1 172.16.100.6:80 check
  server srv2 172.16.100.7:80 check				

listen http-in               # listen 同时定义前后端
  bind *:3306
  server server1 127.0.0.1:3396 maxconn 32
```
