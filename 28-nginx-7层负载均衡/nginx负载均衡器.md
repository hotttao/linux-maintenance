# 28.4 nginx实现7层负载upstream实战

## 1. ngx_http_upstream_module
### 1.1 定义 upstream 组
- 作用: 定义服务器组
- 应用: 可被 proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, and grpc_pass 使用

```
# 1. upstream 上下文
Syntax:    upstream name { ... }
Default:    —
Context:    http

# 2. upstream 内定义服务器组
Syntax:    server address [parameters];
Default:    —
Context:    upstream

parameters:
    weight=number
    max_conns=number
    max_fails=number
    fail_timeout=time
    backup
    down
    slow_start=time
    .......    


# eg： upstream 配置示例
upstream dynamic {
    ip_hash;
    zone upstream_dynamic 64k;

    server backend1.example.com      weight=5;
    server backend2.example.com:8080 fail_timeout=5s slow_start=30s;
    server 192.0.2.1                max_fails=3;
    server backend3.example.com      resolve;
    server backend4.example.com      service=http resolve;

    server backup1.example.com:8080  backup;
    server backup2.example.com:8080  down;
    server unix:/tmp/backend3;
}

server {
    location / {
        proxy_pass http://dynamic;
        health_check;
    }
}


# 3. 激活 nginx 与后端 upstream server 之间的持久连接功能
# 一般非 http 服务器时才会启用
Syntax:    keepalive connections; # 最少连接数
Default:    —
Context:    upstream

# eg: memcache 启动keepalive 长连接
upstream memcached_backend {
    server 127.0.0.1:11211;
    server 10.0.0.2:11211;

    keepalive 32;
}

server {
    ...

    location /memcached/ {
        set $memcached_key $uri;
        memcached_pass memcached_backend;
    }

}


# 4. 健康状态检测
# upstream 内会自动对服务器组进行健康状态检测，但检测的是服务是否存在
# health_check 可在特定应用内，按照特定的方式进行额外的健康状态检测
# 建议在此location 中关闭访问日志
Syntax:    health_check [parameters];
Default:    —
Context:    location

parameters
    interval=time
    uri=uri
    match=name

# eg: 健康状态检测
http {
    server {
    ...
        location / {
            proxy_pass http://backend;
            health_check match=welcome;
        }
    }

    match welcome {
        status 200;
        header Content-Type = text/html;
        body ~ "Welcome to nginx!";
    }
}
```


### 1.2 upstream 负载均衡调度算法
```
# 调度算法
ip_hash;    # 基于源 ip 的session 绑定机制
least_conn  # 最少连接，相当于 lvs wlc 算法
least_time
queue
sticky      # 基于 cookie的 session 绑定
sticky_cookie_insert  # 同 sticky, 在老版 nginx 总被使用


# 1. sticky 实现 session 绑定
Syntax:    sticky cookie name [expires=time] [domain=domain] [httponly] [secure] [path=path];
sticky route $variable ...;
sticky learn create=$variable lookup=$variable zone=name:size [timeout=time] [header] [sync];
Default:    —
Context:    upstream

## 1.1 cookie
upstream backend {
    server backend1.example.com;
    server backend2.example.com;

    sticky cookie srv_id expires=1h domain=.example.com path=/;
}

## 1.2 route
map $cookie_jsessionid $route_cookie {
    ~.+\.(?P<route>\w+)$ $route;
}

map $request_uri $route_uri {
    ~jsessionid=.+\.(?P<route>\w+)$ $route;
}

upstream backend {
    server backend1.example.com route=a;
    server backend2.example.com route=b;

    sticky route $route_cookie $route_uri;
}

## 1.3 learn
upstream backend {
  server backend1.example.com:8080;
  server backend2.example.com:8081;

  sticky learn
          create=$upstream_cookie_examplecookie
          lookup=$cookie_examplecookie
          zone=client_sessions:1m;
}
```

### 1.3 ngx_http_upstream_module 提供的内置变量
```
$upstream_addr              # 挑选的上游服务器地址
$upstream_cache_status      # 缓存命中状态

# 自定义响应首部
http {
    add_header X-Via $server_addr;
    add_header X-Cache $upstream_cache_status
}
```
