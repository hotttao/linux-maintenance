# 26.5 nginx http 高级配置
本节是 nginx http 配置的第三部分，http 高级配置，包括:
1. 网络连接相关的设置
2. 客户端请求的限制
3. 文件操作的优化
4. 客户端请求的特殊处理
5. 内存及磁盘资源分配


## 1. http 高级配置
### 1.1 网络连接相关的设置
```
keepalive_timeout time;
# 保持连接的超时时长；默认为75秒；

keepalive_requests n;
# 在一次长连接上允许承载的最大请求数；

keepalive_disable [msie6 | safari | none ]
# 对指定的浏览器(User Agent)禁止使用长连接；

tcp_nodelay on|off
# 对keepalive连接是否使用TCP_NODELAY选项, on 表示不启用；

client_header_timeout time;
# 读取http请求首部的超时时长；

client_body_timeout time;
# 读取http请求包体的超时时长；

send_timeout time;
# 发送响应的超时时长；
```

### 1.2 对客户端请求的限制
```
limit_except method ... { ... }
# 指定对范围之外的其它方法的访问控制；
limit_except GET {
    allow 172.16.0.0/16;
    deny all;
}

client_max_body_size SIZE;
# http请求包体的最大值；
# 常用于限定客户所能够请求的最大包体；
# 根据请求首部中的Content-Length来检测，以避免无用的传输；

limit_rate speed;
# 限制客户端每秒钟传输的字节数；默认为0，表示没有限制；

limit_rate_after time;
# nginx向客户发送响应报文时，如果时长超出了此处指定的时长，则后续的发送过程开始限速；
```

### 1.3 文件操作的优化
```
sendfile on|off
# 是否启用sendfile功能；

aio on|off
# 是否启用aio功能；

open_file_cache max=N [inactive=time]|off
# 作用: 是否打开文件缓存功能；
# 参数：
#      max: 缓存条目的最大值；当满了以后将根据LRU算法进行置换；
#      inactive: 某缓存条目在指定时长时没有被访问过时，将自动被删除；默认为60s;
# 缓存信息包括：
#      文件句柄、文件大小和上次修改时间；
#      已经打开的目录结构；
#      没有找到或没有访问权限的信息；

open_file_cache_errors on|off
# 是否缓存文件找不到或没有权限访问等相关信息；

open_file_cache_valid time;
# 多长时间检查一次缓存中的条目是否超出非活动时长，默认为60s;

open_file_cache_min_use n;
# 在inactive指定的时长内被访问超此处指定的次数地，才不会被删除；
```

### 1.4 客户端请求的特殊处理
```
ignore_invalid_headers on|off
# 是否忽略不合法的http首部；默认为on;
# off意味着请求首部中出现不合规的首部将拒绝响应；只能用于server和http;

log_not_found on|off
# 是否将文件找不到的信息也记录进错误日志中；

resolver address;
# 指定nginx使用的dns服务器地址；

resover_timeout time;
# 指定DNS解析超时时长，默认为30s;

server_tokens on|off;
# 是否在错误页面中显示nginx的版本号；
```

### 1.5 内存及磁盘资源分配
```
client_body_in_file_only on|clean|off
# HTTP的包体是否存储在磁盘文件中；
# 非off表示存储，即使包体大小为0也会创建一个磁盘文件；
# on表示请求结束后包体文件不会被删除，clean表示会被删除；

client_body_in_single_buffer on|off;
# HTTP的包体是否存储在内存buffer当中；默认为off；

cleint_body_buffer_size size;
# nginx接收HTTP包体的内存缓冲区大小；

client_body_temp_path dir-path [level1 [level2 [level3]]];
client_body_temp_path /var/tmp/client/  1 2
# HTTP包体存放的临时目录；

client_header_buffer_size size;
# 正常情况下接收用户请求的http报文header部分时分配的buffer大小；默认为1k;

large_client_header_buffers number size;
# 存储超大Http请求首部的内存buffer大小及个数；

connection_pool_size size;
# nginx对于每个建立成功的tcp连接都会预先分配一个内存池，
# 此处即用于设定此内存池的初始大小；默认为256；

request_pool_size size;
# nginx在处理每个http请求时会预先分配一个内存池，此处即用于设定此内存池的初始大小；默认为4k;
```
