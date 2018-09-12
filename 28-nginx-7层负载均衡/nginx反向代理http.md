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
1. 如果存在 url 重定向，会重定向之后的 url 进行匹配
2. url_pattern 是 `~/~*` 表示的正则表达式 则 URL不能包含 PATH，否则为语法错误，即 URL 只能是 `http[s]://IP:PORT`
3. 默认情况下，nginx 会将 location 匹配到的剩余部分直接附加在 URL 后，传递给后端服务器，所以 locations 与 URL 通常是要么都以 `/` 结尾，要么都不以 `/` 结尾。
4. 特殊的，如果 URL 的 PATH 仅仅为 `/`,`/` 有没有均可

```
# 2. 正式表达式，URL 不能有 PATH
# a.jpg --> http://192.168.0.10/a.jpg
location ~* \.(jpg|png|gif)$ {
    proxy_pass http://192.168.0.10;
    # proxy_pass http://192.168.0.10/;        # 错误
    # proxy_pass http://192.168.0.10/images/; # 错误
    # proxy_pass http://192.168.0.10/images;  # 错误  
}


# 3.
# /admin/status.php --> /pamstatus.php
# status.php 被直接附加在 URL 后
location /admin/ {
    proxy_pass http://192.168.0.10/pam
}

# 3.
# URL 只以 / 结尾时，/ 是可省略的
location / {
    proxy_pass http://192.168.0.10;
    proxy_pass http://192.168.0.10/;
}
```

### 2.2 后端服务配置
#### proxy_cache_path
`proxy_cache_path path options`
- Default:    —
- Context:    http
- 作用: 定义缓存路经及属性
- options:
  - `[levels=levels]`
  - `keys_zone=name:size`
  - `[inactive=time]`
  - `[max_size=size]`

#### proxy_cache
`proxy_cache zone | off`
- Default:    proxy_cache off;
- Context:    http, server, location
- 作用: 使用缓存
- 参数:
  - `zone`: proxy_cache_path 定义的缓存

#### proxy_cache_valid
`proxy_cache_valid [code ...] time`;
- Default:    —
- Context:    http, server, location
- 作用: 定义不同响应码的缓存时长
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

### 1.3 超时时长设置
```
proxy_connect_timeout  # 设置连接被代理服务器的超时时长
proxy_read_timeout     #
proxy_send_timeout     #
proxy_hide_header      # 隐藏由被代理服务器响应给客户端的指定首部
```
