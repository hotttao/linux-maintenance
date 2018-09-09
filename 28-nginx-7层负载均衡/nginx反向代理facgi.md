# 28.2 nginx反向代理facgi
在讲解 httpd 的时候，我们说过通过 php 搭建一个 动态站点时，httpd 与 php 有三种结合方式
1. CGI: 由 httpd 服务创建子进程来加载和执行 php 脚本
2. fpm（FastCGI Process Manager): php 进程管里器，将 php 的解析执行作为独立的应用程序服务器
3. modules: 将 php编译成为 httpd 的模块，httpd 既是 web 服务器也是应用程序服务器

nginx 与 php 结合的话则只能通过 fpm，将 php 运行为独立的应用程序服务器，nginx 通过反代的模式与 fpm 结合起来。nignx 基于 ngx_http_fastcgi_module 模块就能作为 fastcgi 协议的客户端与 fpm 通信。本节我们就来详解 nignx fastcgi 反向代理的相关配置。

## 1. ngx_http_fastcgi_module
ngx_http_fastcgi_module 提供的配置的参数与 ngx_http_proxy_module 提供的参数几乎完全相同，只是将开头的 http 换成的 fastcgi。

### 1.1 fastcgi 反向代理配置
```
# LNMP 配置
# 1. 安装 php-fpm, php-mysql
yum install php-fpm
rpm -ql php-fpm

yum install php-mysql
rpm -ql php-mysql

#  2. 配置 nginx
location / {
    root /user/share/nginx/html;
    fastcgi_pass  localhost:9000;
    fastcgi_index index.php;

    include      fastcgi_params;
}

# 编辑/etc/nginx/fastcgi_params，将其内容更改为如下内容：
fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx;
fastcgi_param  QUERY_STRING      $query_string;
fastcgi_param  REQUEST_METHOD    $request_method;
fastcgi_param  CONTENT_TYPE      $content_type;
fastcgi_param  CONTENT_LENGTH    $content_length;
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI      $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;
```

### 1.2 fastcgi 缓存配置
参数类似 proxy_cache
```
http {
    fastcgi_cache_path /data/nginx/cache keys_zone=cache_zone:10m inactive=3m max_size=1g;

    map $request_method $purge_method {
        PURGE  1;
        default 0;
    }

    server {
        ...
        location / {
            fastcgi_pass        backend;
            fastcgi_cache      cache_zone;
            fastcgi_cache_key  $uri;
            fastcgi_cache_purge $purge_method;
        }
    }
}
```
