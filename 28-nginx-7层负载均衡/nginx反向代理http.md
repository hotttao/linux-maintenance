# 28.1 nginx反向代理http
nginx 是高度模块化，http 的反向代理功能主要由 `ngx_http_proxy_module` 模块提供，本节我们来讲解如何将 nginx 配置成一个 http 的反向代理服务器，内容包括:
1. nginx 七层反向代理原理
2. 反向代理服务器参数配置
  - 后端服务配置
  - 代理缓存配置
  - http 首部字段配置
  - 超时时长配置


## 1. nginx 七层反向代理原理

## 2. ngx_http_proxy_module
```
# http 反向代理示例
location / {
    proxy_pass       http://localhost:8000;
    proxy_set_header Host      $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### 2.1 后端服务配置
#### proxy_pass
`proxy_pass URL`
- 作用: 指定被代理的后端服务器
- 参数: `URL=http://IP:PORT[/PATH]`
- Default:	—
- Context:	location, if in location, limit_except

```
locations url_pattern{
    proxy_pass URL
}
```
nginx 通过 proxy_pass URL 传递 location 匹配到的 url 时存在一些规则和限制
1. proxy_pass后面的路径不带uri时，其会将location的uri传递给后端主机；
2. proxy_pass后面的路径是一个uri时，其会将location的uri替换为proxy_pass的uri，效果是 nginx 会将 location 匹配到的剩余部分直接附加在 URL 后，传递给后端服务器，所以 locations 与 URL 通常是要么都以 `/` 结尾，要么都不以 `/` 结尾。
5. 如果location定义其uri时使用了正则表达式的模式，或在if语句或limt_execept中使用proxy_pass指令，则proxy_pass之后必须不能使用uri; 用户请求时传递的uri将直接附加代理到的服务的之后；


```
# 1.
location /uri/ {
    proxy http://hos[:port];
}
http://HOSTNAME/uri --> http://host/uri


# 2.
location /uri/ {
  proxy http://host/new_uri/;
}
http://HOSTNAME/uri/ --> http://host/new_uri/

location /uri/ {
  proxy http://host/new_uri;   # 错误，要么都以 `/` 结尾，要么都不以 `/` 结尾。
}
http://HOSTNAME/uri/test --> http://host/new_uritest

# 3.
location ~|~* /uri/ {
  proxy http://host;
}
http://HOSTNAME/uri/ --> http://host/uri/；
```

### 2.2 代理缓存配置
#### proxy_cache_path
`proxy_cache_path path options`
- Default:    —
- Context:    http
- 作用: 定义可用于proxy功能的缓存
- options:
  - `[levels=levels]`: 缓存的目录结构层级
  - `keys_zone=name:size`: 缓存区域名称即内存大小
  - `[inactive=time]`: 非活动链接的检测时间间隔
  - `[max_size=size]`: 缓存的文件所占用的最大磁盘大小

#### proxy_cache
`proxy_cache zone | off`
- Default:    proxy_cache off;
- Context:    http, server, location
- 作用: 指明要调用的缓存，或关闭缓存机制
- 参数:
  - `zone`: proxy_cache_path 定义的缓存

#### proxy_cache_key
`proxy_cache_key string`
- Default: `proxy_cache_key $scheme$proxy_host$request_uri;`
- Context:	http, server, location
- 作用: 缓存中用于“键”的内容；

#### proxy_cache_valid
`proxy_cache_valid [code ...] time`;
- Default:    —
- Context:    http, server, location
- 作用: 定义对特定响应码的响应内容的缓存时长；
- 参数:
  - `code`: 响应码
  - `time`: 缓存时长

#### proxy_cache_methods
`proxy_cache_methods GET | HEAD | POST ...`
- Default:    proxy_cache_methods GET HEAD;
- Context:    http, server, location
- 作用: 只对哪些方法获取的内容进行缓存

#### proxy_cache_use_stale
`proxy_cache_use_stale param`
- Default:    proxy_cache_use_stale off;
- Context:    http, server, location
- 作用: 被代理服务器响应失败时，是否使用过期缓存进行响应
- 参数: 可选值包括 `error | timeout | invalid_header | updating | http_500 | http_502 | http_503 | http_504 | http_403 | http_404 | http_429 | off ...;`

#### proxy_cache_min_uses
`proxy_cache_min_uses number`
- Default:    proxy_cache_min_uses 1;
- Context:    http, server, location
- 作用: proxy_path 定义的 inactive 非活动时间内，最少被访问多少次才不会被清理

#### proxy_cache_bypass
`proxy_cache_bypass string ...`
- Default:    —
- Context:    http, server, location
- 作用: 在何种情况下，nginx 将不从缓存中取数据

```
# 缓存配置示例
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

### 2.3 代理 header 设置
#### proxy_set_header
`proxy_set_header field value`
- Default:
  - `proxy_set_header Host $proxy_host;`
  - `proxy_set_header Connection close;`
- Context:	http, server, location
- 作用: 设定发往后端主机的请求报文的请求首部的值

```
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
```

#### proxy_hide_header
`proxy_hide_header field;`
- Default:	—
- Context:	http, server, location
- 作用: 禁止 nginx 将哪些从后端服务器接收的响应传递给客户端，默认情况下 nignx 已经禁止将  “Date”, “Server”, “X-Pad”, and “X-Accel-...” 发送给客户端，此选项的配置值会附加到禁止列表中。

### 2.4 超时设置
#### proxy_connect_timeout
`proxy_connect_timeout time;``
- Default: `proxy_connect_timeout 60s;`
- Context:	http, server, location
- 作用: 与后端服务器建立链接的超时时长

#### proxy_read_timeout
`proxy_read_timeout time;`
- Default: `proxy_read_timeout 60s;`
- Context:	http, server, location
- 作用: nginx 向接收后端服务器响应时，两次报文之间的超时时长

####	proxy_send_timeout
`proxy_send_timeout time;`
- Default: `proxy_send_timeout 60s;`
- Context:	http, server, location
- 作用: nginx 向后端服务器发送请求时，两次报文之间的超时时长


### 3. ngx_http_headers_module
ngx_http_headers_module 允许 nginx 配置发给用户的响应报文的 header

#### add_header
`add_header name value [always];`
- Default:	—
- Context:	http, server, location, if in location
- 作用: 向响应报文中添加自定义首部；

```
add_header X-Via $server_addr;
add_header X-Accel $server_name;
```

#### expires
`expires [modified] time;`  
`expires epoch | max | off;`  
- Default: expires off;
- Context:	http, server, location, if in location
- 作用: 用于定义Expire或Cache-Control首部的值，或添加其它自定义首部；
