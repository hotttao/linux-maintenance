# 26.4 nginx_http配置段(核心模块)
本节我们来讲解 http 配置的第二部分，http 配置段的配置。nginx 没有中心主机的概念，所以 web 服务包括默认的都是虚拟主机，直接支持基于ip，端口和主机名的虚拟主机。http 配置段用于配置 ngnix 的 web 服务器，由`ngx_http_core_codule` 和其他众多的 http 功能模块组成。本节我们首先来讲解 `http_core_codule` 提供的配置指令，内容包括
1. http 配置段简介
  - 配置框架
  - 内置变量
  - if 上下文
2. http 基础配置
  - 虚拟主机定义
  - 访问路径配置
  - 索引及错误页配置
  - 网络连接相关的设置
  - 客户端请求的限制
  - 文件操作的优化
  - 客户端请求的特殊处理
  - 内存及磁盘资源分配

## 1. http 配置段简介
### 1.1 http 配置段框架
http配置段的框架如下所示，其遵循以下一些原则
1. 必须使用虚拟机来配置站点；每个虚拟主机使用一个`server {}`段配置；
2. 非虚拟主机的配置或公共配置，需要定义在`server`之外，`http`之内；
3. 与 http 相关的指令仅能够放置于 http、server、location、upstream、if 上下文，某些指令只能用于 5 中上下文中的某一种

```        
http {
    sendfile            on; # 各server的公共配置
    tcp_nopush          on;
    ...：

    upstream {              # 配置反向代理
    ......
    }

    server {    # 定义一个虚拟主机；nginx支持使用基于主机名或IP的虚拟主机
                # 每个 server 类似于 httpd 中的一个 <virtualHost>
          listen;
          server_name;
          root;
          alias;

          # 类似 http 中的 <location>, 用于定义URL与本地文件系统的关系
          location URL {  
              if  .... {
                  ....
              }
          }
    }
    server {

    }
    ...
}
```

### 1.2 http核心模块的内置变量
http核心模块常用的内置变量:
- `$uri`: 当前请求的uri，不带参数；
- `$request_uri`: 请求的uri，带完整参数；
- `$host`:
    - http请求报文中host首部；
    - 如果请求中没有host首部，则以处理此请求的虚拟主机的主机名代替；
- `$hostname`: nginx服务运行在的主机的主机名；
- `$remote_addr`: 客户端IP
- `$remote_port`: 客户端Port
- `$remote_user`: 使用用户认证时客户端用户输入的用户名；
- `$request_filename`: 用户请求中的URI经过本地root或alias转换后映射的本地的文件路径；
- `$request_method`: 请求方法
- `$server_addr`: 服务器地址
- `$server_name`: 服务器名称
- `$server_port`: 服务器端口
- `$server_protocol`: 服务器向客户端发送响应时的协议，如http/1.1, http/1.0
- `$scheme`:
    - 在请求中使用scheme,
    - 如`https://www.magedu.com/`中的https；
- `$http_HEADER`:
    - 匹配请求报文中指定的HEADER，
    - `$http_host`匹配请求报文中的host首部
- `$sent_http_HEADER`:
    - 匹配响应报文中指定的HEADER，
    - 例如`$http_content_type`匹配响应报文中的content-type首部；
- `$document_root`:当前请求映射到的root配置；

### 1.3 if 上下文
http 配置段内的 `if 上下文`可以根据 nginx 内的变量执行判断逻辑
- 语法: `if (condition) {.....}`
- 应用: 可应用在 server, location 上下文中
- condition: 基于 nginx 变量的测试表达式，支持以下几种测试方式
  1. 变量是否为空: 变量名为空串，或者以"0"开始，为 false，否则为 true
  2. 比较操作: `=  ！=`
  3. 正则表达式模式匹配
        - `~`: 区分大小写
        - `~*`: 不区分大小写
        - `!~ !~*`: 表示取反
  4. 测试是否为文件: `-f` `!-f`
  5. 测试是否为目录: `-d`
  6. 测试文件存在性: `-e`
  7. 检查文件是否有执行权限: `-x`

```
# if 使用示例
server {
    if ($http_user_agent ~* MSIE) {
        rewrite ^(.*)$ /mise/$1 break;
    }
}
```

## 2. http 服务配置
nginx 中所有路经都是 uri，nginx 会按照配置的 uri，按照当前配置文件重新查找一次，以找到匹配的文件。
### 2.1 虚拟主机定义
```    
server {
    listen 8080 default_server;
    server_name www.httttao.com;
    root "/vhost/web/";
}
# 作用: 定义一个虚拟主机
# 特性: nginx支持使用基于主机名或IP的虚拟主机


listen PORT|address[:port]|unix:/PATH/TO/SOCKET_FILE
listen address[:port] [default_server] [ssl] [http2 | spdy]  
       [backlog=number] [rcvbuf=size] [sndbuf=size]  
# 作用: 指定监听的地址和端口
# 参数:
#     default_server：设定为默认虚拟主机；
#     ssl：限制仅能够通过ssl连接提供服务；
#     backlog=number：后援队列长度；
#     rcvbuf=size：接收缓冲区大小；
#     sndbuf=size：发送缓冲区大小；

server_name [...];
# 作用: 指明虚拟主机的主机名称；后可跟多个由空白字符分隔的字符串；
# 过程: 当nginx收到一个请求时，会取出其首部的server的值，而后跟众server_name进行比较，
#      * 匹配任意长度字符串
#      ~ 正则表达式模式匹配
# 主机名匹配顺序如下：
#    先做精确匹配；www.magedu.com
#    左侧通配符匹配；*.magedu.com
#    右侧通配符匹配；www.abc.com, www.*    
#    正则表达式匹配: ~^.*\.magedu\.com\$

server_name_hash_bucket_size 32|64|128;
# 作用: 为了实现快速主机查找，nginx使用hash表来保存主机名；   

tcp_nodelay on|off;
# 在keepalived模式下的连接是否启用TCP_NODELAY选项，建议启用；

tcp_nopush on|off;
# 在sendfile模式下，是否启用TCP_CORK选项，建议启用；

sendfile on | off;
# 是否启用sendfile功能，建议启用；             
```

### 2.2 访问路径配置
```
root path
# 作用:
#    设置资源路径映射
#    用于指明请求的 URL 所对应的资源所在的文件系统上的起始路径
# 位置：http, server, location, if in location；

alias path
# 作用: 只能用于location中，定义路径别名；
# 对比:
#    root path 用于替换 location URL 中 URL 最左侧的 "/"
#    alias path 用于替换 location URL 中 URL 最右侧的 "/"
location /image {
    root '/vhost/web1''
}
http://www.tao.com/images/a.jpg ---> /vhost/web1/images/a.jpg

location /imapge {
    alias "/www/pictures";
}
http://www.tao.com/images/a.jpg ---> /www/pictures/a.jpg

########################################
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
# 功能：
#    允许根据用户请求的URI来匹配指定的各location以进行访问配置；
#    url被匹配到时，将被location块中的配置所处理；
# 匹配类型
#      =：精确匹配；
#      ~：正则表达式模式匹配，匹配时区分字符大小写
#      ~*：正则表达式模式匹配，匹配时忽略字符大小写
#      ^~: URI前半部分匹配，不检查正则表达式
#      不带符号：匹配起始于此uri的所有的url；
# 匹配优先级：精确匹配(=) ^~  ~  ~*  不带任何符号的 location
http {
    server {
        listen 80;
        server_name www.tao.comm;
        location / {
            root '/vhosts/web1';
        }

        location /images/ {
            root '/vhosts/images';
        }

        location ~* \.php$ {
            fcgipass
        }
    }  
}
```

### 2.2 索引及错误页配置
```
index uri ...;
# 作用: 定义默认页面，可参跟多个值；
# 说明: 这里 uri 与 root，文件系统没有任何关系，
#      nginx 会按照当前配置的匹配逻辑，找到 uri 的位置

error_page code ... [=code] uri | @name;
# 作用:
#    错误页面重定向
#    根据 http 响应状态码来指明特用的错误页面
# [=code]: 以指明的响应吗进行响应，而是默认的原来的响应
error_page   500 502 503 504  =200 /50x.html;
location = /50x.html {
    root   html;
}

try_files path1 [path2 ...] uri;
# 作用:
#    自左至右尝试读取由path所指定路径，在第一次找到即停止并返回
#    如果所有path均不存在，则返回最后一个uri;                
# eg:
# location ~* ^/documents/(.*)\$ {
#    root /www/htdocs;
#    try_files \$uri /docu/\$1 /temp.html;
# }

# http://www.magedu.com/documents/a.html
# http://www.magedu.com/docu/a.html
# http://www.magedu.com/temp.html
```


## 2.3 网络连接相关的设置
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

client_body_buffer_size size;
# 用于接收客户端请求报文的body部分的缓冲区大小；默认为16k；
# 超出此大小时，其将被暂存到磁盘上的由client_body_temp_path指令所定义的位置；

client_body_temp_path path [level1 [level2 [level3]]];
# 设定用于存储客户端请求报文的body部分的临时存储路径及子目录结构和数量；
client_body_temp_path   /var/tmp/client_body  2 1 1
#  1：表示用一位16进制数字表示一级子目录；0-f
#  2：表示用2位16进程数字表示二级子目录：00-ff
#  2：表示用2位16进程数字表示三级子目录：00-ff
```

### 2.4 对客户端请求的限制
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

### 2.5 文件操作的优化
```
sendfile on|off
# 是否启用sendfile功能；

aio on | off | threads[=pool];
# 是否启用aio功能；

directio size | off;
# 在Linux主机启用O_DIRECT标记，此处意味文件大于等于给定的大小时使用，例如directio 4m;

open_file_cache max=N [inactive=time]|off
# 作用: 是否打开文件缓存功能；
# 参数：
#      max: 缓存条目的最大值；当满了以后将根据LRU算法进行置换；
#      inactive: 缓存项的非活动时长，某缓存条目在此选项指定时长内没有被访问过或
#                访问次数少于open_file_cache_min_uses指令所指定的次数，则为非活动项，
#                将自动被删除；默认为60s;
# 缓存信息包括：
#      文件句柄、文件大小和上次修改时间；
#      已经打开的目录结构；
#      没有找到或没有访问权限的信息；

open_file_cache_min_uses number;
# 在open_file_cache指令的inactive参数指定的时长内，至少应该被命中多少次方可被归类为活动项；

open_file_cache_valid time;
# 多长时间检查一次缓存中的条目是否超出非活动时长，默认为60s;

open_file_cache_errors on|off
# 是否缓存文件找不到或没有权限访问等相关信息；
```

### 2.6 客户端请求的特殊处理
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

### 2.7 内存及磁盘资源分配
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
