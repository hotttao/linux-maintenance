# 28.2 nginx反向代理facgi

## 1. ngx_http_fastcgi_module
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
