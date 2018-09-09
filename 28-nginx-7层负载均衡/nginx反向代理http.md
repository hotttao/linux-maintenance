# 28.1 nginx反向代理http

## 1. ngx_http_proxy_module 模块
### 1.1 proxy_pass
功能: http 方向代理
```
location /url {
    proxy_pass      http://back_server:port/newurl;
    proxy_set_header Host      $host;
    proxy_set_header X-Real-IP $remote_addr;
}
# 示例-使用正则表达式
# a.jpg --> http://192.168.0.10/a.jpg
location ~* \.(jpg|png|gif)$ {
    proxy_pass http://192.168.0.10;
    # proxy_pass http://192.168.0.10/;        # 错误
    # proxy_pass http://192.168.0.10/images/; # 错误
    # proxy_pass http://192.168.0.10/images;  # 错误  
}
```
url代理规则:
1. 使用非匹配模式的url，直接请求 proxy_pass 映射的 new_url
2. 存在 url 重定向，使用重定向之后的 url 进行匹配
3. 使用 `~/~*/^~` 进行模式匹配的 url，/new_url 不能存在，否则存在语法错误，代理映射的连接为 http://back_server:port/url


### 1.2 proxy_cache
- 功能: 代理缓存
```
# 1. proxy_cache_path: 定义缓存路径
Syntax:    proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default:    —
Context:    http

# 2. 缓存的使用
Syntax:    proxy_cache zone | off;
Default:    
proxy_cache off;
Context:    http, server, location

# 3. 定义不同响应码的缓存时长
Syntax:    proxy_cache_valid [code ...] time;
Default:    —
Context:    http, server, location

# 4. 只对哪些方法获取的内容进行缓存
Syntax: proxy_cache_methods GET | HEAD | POST ...;
Default:    proxy_cache_methods GET HEAD;
Context:    http, server, location

# 5. 被代理服务器响应失败时，是否使用过期缓存进行响应
Syntax: proxy_cache_use_stale error | timeout | invalid_header | updating | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | http_429 | off ...;
Default:    proxy_cache_use_stale off;
Context:    http, server, location

# 6. 最少被请求多少次才会被缓存
Syntax: proxy_cache_min_uses number;
Default:    proxy_cache_min_uses 1;
Context:    http, server, location

# 7. 在何种情况下，nginx 将不从缓存中取数据
Syntax: proxy_cache_bypass string ...;
Default:    —
Context:    http, server, location


# eg: ------------------------------------------------------------
# 缓存条目: /data/nginx/cache/c/29/b7f54b2df7773722d382f4809d65029c
http {
    proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=one:10m; # 配置
    server {
        ...
        location / {
            proxy_pass http://backend;
            proxy_cache cache_zone;            # 使用
            proxy_cache_key $uri;

            proxy_cache_valid 200 302 10m;
            proxy_cache_valid 301      1h;
            proxy_cache_valid any      1m;

            proxy_cache_use_stale error timeout http_500 http_502 http_503;

            proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment;
            proxy_cache_bypass $http_pragma    $http_authorization;

        }
    }
}

```

### 1.3 其他相关配置
```
proxy_connect_timeout  # 设置连接被代理服务器的超时时长
proxy_read_timeout    #
proxy_hide_header      # 隐藏由被代理服务器响应给客户端的指定首部
```
