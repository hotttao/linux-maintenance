# 28.3 nginx负载均衡器
nginx http 的负载均衡功能主要由 ngx_http_upstream_module 提供，其作用是将后端服务器定义为 nginx 中的服务器组，然后将服务器组作为反向代理的目标。在反代请求时，服务器组首先通过配置的调度算法选择一个后端服务器，然后向其转发请求。因此服务器组是 nginx 负载均衡功能的核心组件。  

服务器组在 nginx 中是通过 upstream 上下文定义的。upstream 上下文内有特定的配置选项，用于定制后端服务器的权重，调度算法，健康状态检测策略等等。

## 1. ngx_http_upstream_module
### 1.1 定义 upstream 组

`upstream name { ... }`
- Default:    —
- Context:    http
- 作用: upstream 上下文，用于定义服务器组
- 应用: 可被 proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, and grpc_pass 使用


```
upstream httpdsrvs {
  server ...
  server...
  ...
}
```


### 1.2 upstream 服务器配置选项
#### server
`server address [parameters];`  
- Default:    —
- Context:    upstream
- 选项:
  - `weight=num`      : 权重     
  - `max_conns=number`: 最大并发连接数
  - `max_fails=number`: 健康状态监测: 失败多少次将被标记为不可用，为 0 表示不做健康状态检测，默认为 1
  - `fail_timeout=tim`: 健康状态检测: 定义失败的超时时长，默认为 10s
  - `backup`          : 指定备用服务器(sorry server)
  - `down`            : 将服务器标记下线，灰度发布使用
  - `slow_start=time` : 慢启动，指平滑的将请求迁移到新增的服务器上
  - ...其他商用版本参数

```
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
```

#### keepalive
`keepalive connections;`
- Default:    —
- Context:    upstream
- 作用: 激活 nginx 与后端 upstream server 之间的持久连接功能
- 参数: connections 表示每个 workder 进程与后端服务器总共能保持的长连接数

```
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
```

#### health_check
`health_check [parameters];`
- Default:    —
- Context:    location
- 作用: 定义对后端主机的健康状态检测机制；只能用于location上下文；
- 选项:
  - `interval=time`：检测频率，默认为每隔5秒钟；
  - `fails=number`：判断服务器状态转为失败需要检测的次数；
  - `passes=number`：判断服务器状态转为成功需要检测的次数；
  - `uri=uri`：判断其健康与否时使用的uri；
  - `match=name`：基于指定的match来衡量检测结果的成败，name 是 `match` 上下文定义的检测机制
  - `port=number`：使用独立的端口进行检测；
- 说明: 健康状态检测
  - upstream 内会自动对服务器组进行健康状态检测，但检测的是服务是否存在
  - health_check 可在特定应用内，按照特定的方式进行额外的健康状态检测
  - 建议在此location 中关闭访问日志
  - 仅Nginx Plus有效；

```
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

#### match
`match{}` 上下文用来定义衡量某检测结果是否为成功的衡量机制；有如下专用指令：
1. `status`：期望的响应码；
  - `status CODE`
  - `status ! CODE`
2. `header`：基于响应报文的首部进行判断
  - `header HEADER=VALUE`
  - `header HEADER ~ VALUE`
3. body：基于响应报文的内容进行判断
  - `body ~ "PATTERN"`
  - `body !~ "PATTERN"`

需要注意的是 `match{}`, `health_check` 都进在仅Nginx Plus中有效


### 1.2 upstream 调度算法配置
#### least_conn
`least_conn`
- Default:	—
- Context:	upstream
- 作用: 最少连接调度算法； 当server拥有不同的权重时为wlc；当所有后端主机的连接数相同时，则使用wrr进行调度；

#### least_time
`least_time header | last_byte [inflight];`
- Default:	—
- Context:	upstream
- 作用: 最短平均响应时长和最少连接；
- 参数:
  - header：response_header;
  - last_byte: full_response;
- 说明: 仅Nginx Plus有效；

#### ip_hash
`ip_hash;`
- Default:	—
- Context:	upstream
- 作用: 源地址hash算法；能够将来自同一个源IP地址的请求始终发往同一个upstream server；

#### hash
`hash key [consistent];`
- Default:	—
- Context:	upstream
- 作用: 基于指定的key的hash表实现请求调度，此处的key可以文本、变量或二者的组合；
- 参数: consistent 指定使用一致性hash算法；

```
hash $request_uri consistent    # 基于请求 url 进行绑定，lvs 的 DH算法
hash $remote_addr               # == ip_hash
hash $cookie_name
```  

#### sticky
用于实现基于 cookie的 session 绑定，只在商业版才能使用


### 1.3 http upstream 内置变量
内置变量:
- `$upstream_addr`: 挑选的上游服务器地址
- `$upstream_cache_status`: 缓存命中状态

```
# 自定义响应首部
http {
    add_header X-Via $server_addr;
    add_header X-Cache $upstream_cache_status
}
```
