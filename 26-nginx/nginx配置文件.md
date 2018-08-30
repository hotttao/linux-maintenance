# 26.3 nginx配置文件全面讲解
## 2. 基础核心配置
最长需要修改的参数:
- `worker_process`
- `worker_connections`
- `worker_cpu_affinity`
- `worker_priority`

### 2.1 正常运行的必备配置：
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

### 2.2 优化性能相关的配置：
```

worker_processes n;
# worker进程的个数；通常其数值应该为CPU的物理核心数减1；


worker_cpu_affinity cpumask ...;
# 作用: 提升缓存命中率
# eg:
#    worker_processes 6;
#    worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000;


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
### 2.3 事件相关的配置
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
# 作用: 指明使用的时间模型，建议让Nginx 自动选择

work_connections  nums
# 设定单个 worker 进程能处理的最大并发连接数量, 尽可能大，以避免成为限制，eg: 51200

multi_accept on|off;
# 作用: 是否允许一次性地响应多个用户请求；默认为Off;

use [epoll|rtsig|select|poll]
# 作用: 定义使用的事件模型，建议让nginx自动选择；

worker_connections n;
# 作用: 每个worker能够并发响应最大请求数；
```

### 2.4 用于调试、定位问题: 只调试nginx时使用
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

## 3. nginx的 http web功能：
- 组件: http {} 由 ngx_http_core_codule 模块所引入
- 配置:
    1. 必须使用虚拟机来配置站点；每个虚拟主机使用一个server {}段配置；
    2. 非虚拟主机的配置或公共配置，需要定义在server之外，http之内；
    3. 与 http 相关的指令仅能够放置于 http、server、location、upstream、if 上下文，某些指令只能用于 5 中上下文中的某一种
- 配置框架:
```        
http {
    upstream {  # 配置反向代理
    ......
    }

    server {    # 定义一个虚拟主机；nginx支持使用基于主机名或IP的虚拟主机
                # 每个 server 类似于 httpd 中的一个 <virtualHost>
        location URL {  
            root  "/path"
        }
        location URL {  # 类似 http 中的 <location>, 用于定义URL与本地文件系统的关系
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

### 3.1 http 基本及访问路径配置
```    
server {
    listen 8080;
    server_name www.httttao.com;
    root "/vhost/web/";
    default_server
}
# 作用: 定义一个虚拟主机
# 特性: nginx支持使用基于主机名或IP的虚拟主机


listen  address[:port];
listen  port;
# 作用: 指定监听的地址和端口


server_name [...];
# 作用: server_name可以跟多个主机名，名称中可以使用通配符和正则表达式(通常以~开头)
# 过程: 当nginx收到一个请求时，会取出其首部的server的值，而后跟众server_name进行比较，
# 主机名比较顺序如下：
#    先做精确匹配；www.magedu.com
#    左侧通配符匹配；*.magedu.com
#    右侧通配符匹配；www.abc.com, www.*
#    正则表达式匹配: ~^.*\.magedu\.com\$
#    default_server


default_server：
# 作用: 定义此server为http中默认的server
# 附注: 如果所有的server中没有任何一个listen使用此参数，那么第一个server即为默认server


server_name_hash_bucket_size 32|64|128;
# 作用: 为了实现快速主机查找，nginx使用hash表来保存主机名；                


root path
# 作用:
#    设置资源路径映射
#    用于指明请求的 URL 所对应的资源所在的文件系统上的起始路径

alias path
# 作用: 只能用于location中，定义路径别名；
# 对比:
#    root path 用于替换 location URL 中 URL 最左侧的 "/"
#    alias path 用于替换 location URL 中所有的URL
location /image {
    root '/vhost/web1''
}
http://www.tao.com/images/a.jpg ---> /vhost/web1/images/a.jpg

location /imapge {
    alias "/www/pictures";
}
http://www.tao.com/images/a.jpg ---> /www/pictures/a.jpg

####################
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
# 功能：
#    允许根据用户请求的URI来匹配指定的各location以进行访问配置；
#    匹配到时，将被location块中的配置所处理；
# 匹配类型
#      =：精确匹配；
#      ~：正则表达式模式匹配，匹配时区分字符大小写
#      ~*：正则表达式模式匹配，匹配时忽略字符大小写
#      ^~: URI前半部分匹配，不检查正则表达式
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

### 3.2 http 索引及错误页配置
```
index file ...;
# 作用: 定义默认页面，可参跟多个值；

error_page code ... [=code] uri | @name;
# 作用:
#    错误页面重定向
#    根据 http 响应状态码来指明特用的错误页面
# [=code]: 以指明的响应吗进行响应，而是默认的原来的响应
# eg: error_page  404 /404_customed.html

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

### 3.3 http 访问控制
```
allow IP/Network;
deny IP/Netword;
# 作用: 基于 ip 的访问控制, all 表示所有


auth_basic
# 作用: 基于用户的访问控制
auth_basic_user_file
# 作用: 账号密码文件
# 产生: 建议使用 htppasswd 创建

location /admin/ {
    auth_basic "only for adminor";
    auth_basic_user_filer /etc/nginx/user/.httppasswd;      
}
htpasswd -c -m /path/user/.passwd tom
```


### 3.4 https 服务配置
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

### 3.5 http if 上下文
- 语法: if (condition) {.....}
- 应用: 可应用在 server, location 上下文中
- condition:
    - 变量名: 变量名为空穿，或者以"0"开始，则为 false
    - 以变量为操作数的比较表达式: =  ！=
    - 正则表达式的模式匹配操作
        - ~: 区分大小写
        - ~*: 不区分大小写
        - !~ !~* 表示取反
    - 测试是否为文件: -f !-f
    - 测试是否为目录: -d
    - 测试文件存在性: -e
    - 检查文件是否有执行权限: -x

```
server {
    if ($http_user_agent ~* MSIE) {
        rewrite ^(.*)$ /mise/$1 break;
    }
}
# if 使用示例


location ~* \.(jpg|gif|jpeg) {
    valid_referer  none blocked  www.httttao.com;
    if ($invalid_referer) {
        rewrite ^/  http://www.hotttao.com/403.html;
    }
}
# 防盗链

```

### 3.6  其他相关配置
1. stub_status: 是否开启状态页
2. rewrite: url 重写
3. log_format: 自定义日志格式
```
location /status {
  stub_status on;
  allow  172.16.0.0/16;
  deny all;
}
stub_status {on|off};
# 作用: 是否开启状态页；
# 附注: 仅能用于 location 上下文
# 显示:
#    Active connnections: 当前所有处于打开状态的连接数
#    server accept handled requests  n1 n2 n3
#        - n1：表示已经接受的链接
#        - n2：表示已经处理的链接
#        - n3：表示已经处理的请求数
#    Reading: n  Writing: w  Waiting: t
#      - Reading: 正处于接收请求状态爱的链接数
#      - Writing: 请求已经完成，正处于处理请求或发送响应过程中的连接数
#      - Waiting: 保持连接模式，且处于活动状态的连接数


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

## 4. 其他配置
### 4.1 网络连接相关的设置：
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

### 4.2 对客户端请求的限制：
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

### 4.3 文件操作的优化：
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

### 4.4 对客户端请求的特殊处理：
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

### 4.5 内存及磁盘资源分配：
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

## 5. http核心模块的内置变量：
- \$uri: 当前请求的uri，不带参数；
- \$request_uri: 请求的uri，带完整参数；
- \$host:
    - http请求报文中host首部；
    - 如果请求中没有host首部，则以处理此请求的虚拟主机的主机名代替；
- \$hostname: nginx服务运行在的主机的主机名；
- \$remote_addr: 客户端IP
- \$remote_port: 客户端Port
- \$remote_user: 使用用户认证时客户端用户输入的用户名；
- \$request_filename: 用户请求中的URI经过本地root或alias转换后映射的本地的文件路径；
- \$request_method: 请求方法
- \$server_addr: 服务器地址
- \$server_name: 服务器名称
- \$server_port: 服务器端口
- \$server_protocol: 服务器向客户端发送响应时的协议，如http/1.1, http/1.0
- \$scheme:
    - 在请求中使用scheme,
    - 如https://www.magedu.com/中的https；
- \$http_HEADER:
    - 匹配请求报文中指定的HEADER，
    - \$http_host匹配请求报文中的host首部
- \$sent_http_HEADER:
    - 匹配响应报文中指定的HEADER，
    - 例如\$http_content_type匹配响应报文中的content-type首部；
- \$document_root：当前请求映射到的root配置；


## 6. fastcgi 的相关配置
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
