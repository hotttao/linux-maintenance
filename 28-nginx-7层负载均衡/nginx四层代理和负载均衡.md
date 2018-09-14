# 28.4 nginx 四层代理和负载均衡
新版的 nginx 除了代理 http 服务外，还可以基于 stream 模块来实现四层协议的转发、代理或者负载均衡等等。与 LVS 不同的是，nginx 的四层代理依然工作在用户空间，"一手拖两家"，一边作为服务器接收用户请求，另一边作为客户端向后端服务器发送请求。四层的反代和负载均衡配置与 http 的反代和负载均衡基本类似。

## 1. 四层反代和负载均衡示例
```
worker_processes auto;

error_log /var/log/nginx/error.log info;

events {
    worker_connections  1024;
}

stream {
    upstream backend {                      # stream 模块的 upstream 模块
        hash $remote_addr consistent;

        server backend1.example.com:12345 weight=5;
        server 127.0.0.1:12345            max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;
    }

    upstream dns {
       server 192.168.0.1:53535;
       server dns.example.com:53;
    }

    server {
        listen 12345;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }

    server {
        listen 127.0.0.1:53 udp reuseport;
        proxy_timeout 20s;
        proxy_pass dns;
    }

    server {
        listen [::1]:12345;
        proxy_pass unix:/tmp/stream.socket;
    }
}
```

## 2. ngx_stream_core_module
stream 模块使用 `server { ... }` 上下文来配置反代的四层服务，有如下特殊的配置指令

#### listen
`listen address:port options`
- Default:	—
- Context:	server
- 作用: nginx 反代服务监听的地址和端口
- 选项:
  - `[udp]`: 默认为tcp协议, 指定监听udp协议的端口
  - `[backlog=number]`
  - `[bind]`
  - `[ipv6only=on|off]`
  - `[reuseport]`
  - `[so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];`

```
server {
    listen 12345;
    proxy_connect_timeout 1s;
    proxy_timeout 3s;
    proxy_pass backend;
}
```

## 3. ngx_stream_proxy_module
ngx_stream_proxy_module 主要来实现四层代理功能，其作用与提供的指令与 ngx_http_proxy_module 基本一致

#### proxy_pass address;
`proxy_pass address;`
- Default:	—
- Context:	location, if in location
- 作用: 配置后端服务器
- 参数: address 为后端服务器的地址，可以 IP，域名, upstream 定义的服务器组

#### proxy_timeout timeout;
`proxy_timeout timeout;`
- Default: `proxy_timeout 10m;`
- Context:	stream, server
- 作用: 设置 nginx 与客户端和后端服务器，超过多长时间未传输数据时则断开链接

#### proxy_connect_timeout
`proxy_connect_timeout time;``
- Default: `proxy_connect_timeout 60s;`
- Context:	http, server, location
- 作用: 设置nginx与被代理的服务器尝试建立连接的超时时长；默认为60s；

## 4. ngx_stream_upstream_module
ngx_stream_proxy_module 主要来实现四层负载均衡功能，其作用与提供的指令与 ngx_http_upstream_module 基本一致

#### 配置示例
```
stream {
  upstream sshsrvs {
    server 192.168.10.130:22;
    server 192.168.10.131:22;
    hash $remote_addr consistent;
  }

  server {
    listen 172.16.100.6:22202;
    proxy_pass sshsrvs;
    proxy_timeout 60s;
    proxy_connect_timeout 10s;
  }
}
```
