# 26.5 nginx_http配置段()其他模块)
本节是 nginx http 配置的第三部分。上一节我们讲解了 `http_core_codule` 提供的配置指令，本节我们来讲解 nginx http 配置段的功能模块，内容包括:
1. 功能模块
  - 访问控制
  - 开启状态页
  - url重写和自定义日志格式
  - 访问日志配置
  - 文本压缩
2. https 服务配置
3. fastcgi 配置

## 1. 功能模块
### 1.1 访问控制
```
allow IP/Network;
deny IP/Netword;
# 作用: 基于 ip 的访问控制, all 表示所有
# 模块: ngx_http_access_module模块


auth_basic string | off;
# 作用: 基于用户的访问控制
# 模块: ngx_http_auth_basic_module模块

auth_basic_user_file
# 作用: 账号密码文件
# 产生: 建议使用 htppasswd 创建

location /admin/ {
    auth_basic "only for adminor";
    auth_basic_user_filer /etc/nginx/user/.httppasswd;      
}
htpasswd -c -m /path/user/.passwd tom
```

### 1.2 开启状态页
模块: ngx_http_stub_status_module模块

```
location /status {
  stub_status on;
  allow  172.16.0.0/16;
  deny all;
}
stub_status {on|off};
# 作用: 是否开启状态页，用于输出nginx的基本状态信息
# 附注: 仅能用于 location 上下文
# 显示:
#    Active connnections: 当前所有处于打开状态的连接数
#    server accept handled requests  
#      n1     n2      n3
#        - n1：已经接受的客户端请求的总数；
#        - n2：已经处理完成的客户端请求的总数；
#        - n3：客户端发来的总的请求数；
#    Reading: n  Writing: w  Waiting: t
#      - Reading: 处于读取客户端请求报文首部的连接的连接数；
#      - Writing: 处于向客户端发送响应报文过程中的连接数；
#      - Waiting: 处于等待客户端发出请求的空闲连接数；
```

### 1.3 url 重写与自定义日志
2. rewrite: url 重写
3. log_format: 自定义日志格式

```
location / {
    rewriter ^/bbs/(.*)$  /forum/$1 break;
    rewriter ^/bbs/(.*)$  https://www.tao.com/$1 redirect;  
}
rewrite  regex  replacement flag;
# 作用: url 重写
# eg: rewrite  ^/images/(.*\.jpg)$  /imgs/$1 break;
# flag:
#    last:
#        一旦被当前规则匹配并重写后立即停止检查后续的其它rewrite的规则，
#        而后通过重写后的规则重新发起请求,并从头开始执行 rewriter 检查；
#    break:
        一旦被当前规则匹配并重写后立即停止后续的其它rewrite的规则，
        而后通过重写后的规则重新发起请求
        且不会被当前的location 内的任何 rewriter 规则所检查；
#    redirect: 以302临时重定向返回新的URL；
#    permanent: 以301永久重定向返回新的URL；


log_format format_name  "$remote_addr"
# 作用: 自定义日志格式
# format_name: 自定义格式的格式名
access_log  logs/access.log  format_name;
# 作用: 使用自定义格式记录日志
```

### 1.4 日志配置
模块: ngx_http_log_module模块

```
log_format name string ...;
# 作用: string可以使用nginx核心模块及其它模块内嵌的变量；

access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
# 作用: 访问日志文件路径，格式及相关的缓冲的配置；
#    buffer=size: 日志缓冲区大小
#    flush=time

access_log off;
# 作用: 关闭记录日志功能，

open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time];
open_log_file_cache off;
# 作用: 缓存各日志文件相关的元数据信息；
# 参数:    
#    max：缓存的最大文件描述符数量；
#    min_uses：在inactive指定的时长内访问大于等于此值方可被当作活动项；
#    inactive：非活动时长；
#    valid：验正缓存中各缓存项是否为活动项的时间间隔
```

### 1.5 文本压缩
模块: ngx_http_gzip_module：

```
gzip on | off;
# 作用: 是否启用压缩功能

gzip_comp_level level;
# 作用: 设置压缩级别，1-9

gzip_disable regex ...;
# 作用: 对哪些浏览器禁用压缩
# 参数: regex 用于匹配请求报文 "User-Agent" 头信息

gzip_min_length length;
# 作用: 启用压缩功能的响应报文大小阈值；

gzip_buffers number size;
# 作用: 支持实现压缩功能时为其配置的缓冲区数量及每个缓存区的大小；

gzip_proxied off | expired | no-cache | no-store | private |
             no_last_modified | no_etag | auth | any ...;
# 作用: nginx作为代理服务器接收到从被代理服务器发送的响应报文后，在何种条件下启用压缩功能的；
# 选项:
#   off：对代理的请求不启用
#   no-cache, no-store，private：表示从被代理服务器收到的响应报文首部的
#   Cache-Control的值为此三者中任何一个，则启用压缩功能；

gzip_types mime-type ...;
# 作用: 压缩过滤器，仅对此处设定的MIME类型的内容启用压缩功能；


gzip  on;
gzip_comp_level 6;
gzip_min_length 64;
gzip_proxied any;
gzip_types text/xml text/css  application/javascript;
```

### 2. https 服务配置
生成私钥，生成证书签名请求，并获得证书

```
serve {
    listen 443 ssh;
    server_name www.htttao.com;

    ssl_certificate  /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    ssl_session_cache  shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers HIGH:!aNULLL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        root /vhost/web/;
        index index.html index.htm;
    }
}
```

## 3. fastcgi 配置
```
server {
    location ~ \.php$ {
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_pindex index.php;
        fastcgi_param SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        include fastcgi_params;      
    }
}
```
