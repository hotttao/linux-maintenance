# 28.4 nginx负载均衡器
nginx http 的负载均衡功能主要由 ngx_http_upstream_module 提供，其作用是将后端服务器定义为 nginx 中的服务器组，然后将服务器组作为反向代理的目标。在反代请求时，服务器组首先通过配置的调度算法选择一个后端服务器，然后向其转发请求。因此服务器组是 nginx 负载均衡功能的核心组件。  

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
    weight=num          # 权重     
    max_conns=number    # 最大并发连接数
    max_fails=number    # 健康状态监测: 失败多少次将被标记为不可用，为 0 表示不做健康状态检测，默认为 1
    fail_timeout=time   # 健康状态检测: 定义失败的超时时长，默认为 10s
    backup              # 指定备用服务器(sorry server)
    down                # 将服务器标记下线，灰度发布使用
    slow_start=time     # 慢启动，指平滑的将请求迁移到新增的服务器上
    ....商业版额外参数...    


# eg： upstream 配置示例
http{
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
}

# 3. 激活 nginx 与后端 upstream server 之间的持久连接功能
# connections 表示每个 workder 进程与后端服务器总共能保持的长连接数
Syntax:    keepalive connections; # 
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
ip_hash;    # 基于源 ip 的session 绑定机制  ==  hash $remote_addr
least_conn  # 最少连接，相当于 lvs wlc 算法
least_time
queue
sticky      # 基于 cookie的 session 绑定
sticky_cookie_insert  # 同 sticky, 在老版 nginx 总被使用


# 基于绑定的通用调度方法，consistent 表示使用一致性哈希算法
Syntax: hash key [consistent];
Default:    —
Context:    upstream
This directive appeared in version 1.7.2

# eg:
ip_hash == hash $remote_addr  # 基于 ip 地址进行绑定
hash $request  consistent     # 基于请求 url 进行绑定，lvs 的 DH算法

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

## 2. 一致性哈希算法