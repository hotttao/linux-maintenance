# 26.4 nginx http 配置段
本节我们来讲解 http 配置的第二部分，http 配置段的配置。http 配置段由 `ngx_http_core_codule` 模块所引入，用于配置 ngnix 的 web 服务器。nginx 没有中心主机的概念，所以 web 服务包括默认的都是虚拟主机，直接支持基于ip，端口和主机名的虚拟主机。我们将按照如下顺序讲解 http 配置:
1. http 配置段简介
  - 配置框架
  - 内置变量
  - if 上下文
2. http 基础配置
  - 虚拟主机定义
  - 访问路径配置
  - 索引及错误页配置
  - 访问控制
  - 开启状态页
  - url重写和自定义日志格式
3. https 服务配置
4. fastcgi 配置

限于篇幅，有关 ngnix http 的高级配置我们放在一下节来讲解。

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

# 防盗链
location ~* \.(jpg|gif|jpeg) {
    valid_referer  none blocked  www.httttao.com;
    if ($invalid_referer) {
        rewrite ^/  http://www.hotttao.com/403.html;
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

### 2.3 访问控制
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
### 2.4 开启状态页
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
```

### 2.5  url 重写与自定义日志
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

### 3. https 服务配置
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

## 4. fastcgi 配置
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
