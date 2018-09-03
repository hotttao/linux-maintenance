# 26.5 nginx_http配置段(功能模块)
本节是 nginx http 配置的第三部分。上一节我们讲解了 `http_core_codule` 提供的配置指令，本节我们来讲解 http 的各种功能模块提供的配置指令，内容包括:
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
模块: `ngx_http_stub_status_module` 模块

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
模块: `ngx_http_rewrite_module` 模块

```
rewrite  regex  replacement flag;
# 作用: url 重写
# eg: rewrite  ^/images/(.*\.jpg)$  /imgs/$1 break;
# flag:
#    last: 默认
#        一旦被当前规则匹配并重写后立即停止检查后续的其它rewrite的规则，
#        然后用重写后的规则再从头开始执行 rewriter 检查；
#    break:
#        一旦被当前规则匹配并重写后立即停止后续的其它rewrite的规则，
#        而后通过重写后的规则重新发起请求
#        且不会被当前的location 内的任何 rewriter 规则所检查；
#    redirect: 以302临时重定向返回新的URL；
#    permanent: 以301永久重定向返回新的URL；
# 说明:
#     last,break 只会发生在 nginx 内部，不会与客户端交互，客户端收到的是正常的响应
#     redict, permanent 则是直接返回 30x 响应，跨站重写必需使用 redirect或permanent
#     如果 last 发生死循环，nginx 会在循环 10 此之后返回 50x 响应

return code [text];
return code URL;
return URL;
# 作用: 停止进程并返回特定的响应给客户端，非标准的 444 将直接关闭链接，且不会发送响应

rewrite_log on | off;
# 作用: 是否开启重写日志；

# 示例
server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  last;
    return  403;
    ...
}

location / {
    rewriter ^/bbs/(.*)$  /forum/$1 break;
    rewriter ^/bbs/(.*)$  https://www.tao.com/$1 redirect; # http --> https  
}


if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}

if ($request_method = POST) {
    return 405;
}

if ($slow) {
    limit_rate 10k;
}

if ($invalid_referer) {
    return 403;
}
```

### 1.4 日志配置
模块: `ngx_http_log_module` 模块

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
模块: ngx_http_ssl_module模块
```
ssl on | off;
# 作用: 是否 ssl

ssl_certificate file;
# 作用: 当前虚拟主机使用PEM格式的证书文件；

ssl_certificate_key file;
# 作用:当前虚拟主机上与其证书匹配的私钥文件；

ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2];
# 作用: 支持ssl协议版本，默认为后三个；

ssl_session_cache off | none | [builtin[:size]] [shared:name:size];
# 作用: 是否缓存 sll 会话
# 选项:
#     builtin[:size]：使用OpenSSL内建的缓存，此缓存为每worker进程私有；
#     [shared:name:size]：在各worker之间使用一个共享的缓存；
# 说明: 如果用户请求被不同的 worker 处理时，私有缓存可能时效，因此应该使用公共缓存

ssl_session_timeout time;
# 作用: 客户端一侧的连接可以复用ssl session cache中缓存 的ssl参数的有效时长；

server {
  listen 443 ssl;
  server_name www.magedu.com;
  root /vhosts/ssl/htdocs;
  ssl on;
  ssl_certificate /etc/nginx/ssl/nginx.crt;
  ssl_certificate_key /etc/nginx/ssl/nginx.key;
  ssl_session_cache shared:sslcache:20m;  # 1m 空间大约能缓存 4000 个会话
}							
```

## 3. 防盗链
模块: ngx_http_referer_module模块

```
valid_referers none | blocked | server_names | string ...;
# 作用: 定义referer首部的合法可用值；
# 参数:     
#    none：请求报文首部没有referer首部；
#    blocked：请求报文的referer首部没有值；
#    server_names：参数，其可以有值作为主机名或主机名模式；
#    arbitrary_string：直接字符串，但可使用*作通配符；
#    regular expression：被指定的正则表达式模式匹配到的字符串；要使用~打头，例如 ~.*\.magedu\.com；

$invalid_referer
# 作用: 内置的变量，表示当前请求 referer首部是否不符合 valid_referers 定义的规则

    
#配置示例：
valid_referers none block server_names *.magedu.com *.mageedu.com magedu.* mageedu.* ~\.magedu\.;

if($invalid_referer) {
   return http://www.magedu.com/invalid.jpg;
}

valid_referers none blocked server_names
               *.example.com example.* www.example.org/galleries/
               ~\.google\.;
```
