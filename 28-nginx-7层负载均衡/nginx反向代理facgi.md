# 28.2 nginx反向代理facgi
在讲解 httpd 的时候，我们说过通过 php 搭建一个 动态站点时，httpd 与 php 有三种结合方式
1. CGI: 由 httpd 服务创建子进程来加载和执行 php 脚本
2. fpm（FastCGI Process Manager): php 进程管里器，将 php 的解析执行作为独立的应用程序服务器
3. modules: 将 php编译成为 httpd 的模块，httpd 既是 web 服务器也是应用程序服务器

nginx 与 php 结合的话则只能通过 fpm，将 php 运行为独立的应用程序服务器，nginx 通过反代的模式与 fpm 结合起来。nignx 基于 ngx_http_fastcgi_module 模块就能作为 fastcgi 协议的客户端与 fpm 通信。本节我们就来详解 nignx fastcgi 反向代理的相关配置。

## 1. ngx_http_fastcgi_module
ngx_http_fastcgi_module 提供的配置的参数与 ngx_http_proxy_module 提供的参数几乎完全相同，只是将开头的 http 换成的 fastcgi。

### 1.1 fastcgi 反向代理服务配置
#### fastcgi_pass
`fastcgi_pass address;`
- Default:	—
- Context:	location, if in location
- 参数: address为fastcgi server的地址

#### fastcgi_index
`fastcgi_index name;`
- Default:	—
- Context:	http, server, location
- 作用: fastcgi默认的主页资源;

#### fastcgi_param
`fastcgi_param parameter value [if_not_empty];`
- Default:	—
- Context:	http, server, location
- 作用: 设置传递给后端 fastcgi serve 的参数

```
# 配置示例1：
# 前提：配置好fpm server和mariadb-server服务；
location ~* \.php$ {
  root           /usr/share/nginx/html;
  fastcgi_pass   127.0.0.1:9000;
  fastcgi_index  index.php;
  fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html$fastcgi_script_name;
  include        fastcgi_params;
}

# 配置示例2
# 通过/pm_status和/ping来获取fpm server状态信息；
location ~* ^/(pm_status|ping)$ {
  include        fastcgi_params;
  fastcgi_pass 127.0.0.1:9000;
  fastcgi_param  SCRIPT_FILENAME  $fastcgi_script_name;
}
```

### 1.2 fastcgi 缓存配置
#### fastcgi_cache_path
`fastcgi_cache_path path options`
- Default:	—
- Context:	http
- 作用: 定义fastcgi的缓存；缓存位置为磁盘上的文件系统，由path所指定路径来定义；
- 选项:
  - `levels=levels`：缓存目录的层级数量，以及每一级的目录数量；`levels=ONE:TWO:THREE`
  - `keys_zone=name:size`: k/v映射的内存空间的名称及大小
  - `inactive=time`: 非活动时长
  - `max_size=size`: 磁盘上用于缓存数据的缓存空间上限

#### fastcgi_cache
`fastcgi_cache zone | off;`
- Default: `fastcgi_cache off;`
- Context:	http, server, location
- 作用: 调用指定的缓存空间来缓存数据

#### fastcgi_cache_key
`fastcgi_cache_key string;`
- Default:	—
- Context:	http, server, location
- 作用: 定义用作缓存项的key的字符串；

```
fastcgi_cache_key localhost:9000$request_uri;
```

#### fastcgi_cache_methods
`fastcgi_cache_methods GET | HEAD | POST ...;`
- Default: `fastcgi_cache_methods GET HEAD;`
- Context:	http, server, location
- 作用: 为哪些请求方法使用缓存；

#### fastcgi_cache_min_uses
`fastcgi_cache_min_uses number;``
- Default: `fastcgi_cache_min_uses 1;`
- Context:	http, server, location
- 作用: 缓存空间中的缓存项在inactive定义的非活动时间内至少要被访问到此处所指定的次数方可被认作活动项；

#### fastcgi_cache_valid
`fastcgi_cache_valid [code ...] time;``
- Default:	—
- Context:	http, server, location
- 作用: 不同的响应码各自的缓存时长；

#### fastcgi_keep_conn
`fastcgi_keep_conn on | off;`
- Default: `fastcgi_keep_conn off;`
- Context:	http, server, location
- 作用: 是否启动 nginx 于 fastcgi server 之间的长链接

```
示例： fastcgi 缓存配置
http {
  ...
  fastcgi_cache_path /var/cache/nginx/fastcgi_cache levels=1:2:1 keys_zone=fcgi:20m inactive=120s;
  ...
  server {
    ...
    location ~* \.php$ {
      ...
      fastcgi_cache fcgi;
      fastcgi_cache_key $request_uri;
      fastcgi_cache_valid 200 302 10m;
      fastcgi_cache_valid 301 1h;
      fastcgi_cache_valid any 1m;
      ...
    }
    ...
  }
  ...
}
```
