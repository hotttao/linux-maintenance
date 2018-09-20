# 32.2 haproxy 配置
本节开始我们来学习 haproxy 的配置。haproxy 有众多的配置选项，我打算将其分为两个部分，第一部分为haproxy 的全局配置，以及常用的代理段配置，第二部分为 haproxy 的访问控制。本节为的一部分，内容包括
1. global 全局配置
  - 进程及安全管理
  - 性能调整
2. 代理段配置
  - 前端服务配置
  - 后端主机配置
  - haproxy 状态统计及管理页面配置
  - 自定义错误页
  - 首部处理
  - 日志配置

## 1. global配置参数
### 1.1 进程及安全管理
````
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
````

`deamon`
- 作用: 以后端服务的方式运行 haproxy

`log <address> [len <length>] <facility> [max level [min level]]`
- 作用: 定义全局的syslog服务器；最多可以定义两个；

`nbproc <number>`
- 作用: 要启动的haproxy的进程数量
- 默认: 启动一个进程，无需修改

`ulimit-n <number>`
- 作用: 每个haproxy进程可打开的最大文件数
- 默认: haproxy 会自动调整，无需配置


## 1.2 性能调整:
`maxconn <number>`
- 作用: 设定每个haproxy进程所能接受的最大并发连接数；

`maxsslconn <number>`
- 作用: 设定每个haproxy进程所能接受的最大 sll 并发连接数；

`maxconnrate <number>`
- 作用: 设定每个进程每秒种所能创建的最大连接数量；

`maxsessrate <number>`
- 作用: 设定每个进程每秒种所能创建的最大会话数量；

`spread-checks <0..50, in percent>`
- 作用: 设置后端服务器状态检测的速率偏斜，以免同时对所有后端主机进行状态检测，占用太多带宽


## 2. 代理配置段
与 nginx 类似，代理配置段内的参数有受限的应用范围，下面是常用的配置参数

### 2.1 前端服务配置
#### mode
`mode { tcp|http|health }`
- 作用: 定义haproxy的工作模式；
- 参数:
  - tcp: 基于layer4实现代理；可代理mysql, pgsql, ssh, ssl等协议；
  - http: 仅当代理的协议为http时使用；
  - health: 工作为健康状态检查的响应模式，当连接请求到达时回应“OK”后即断开连接；

```
listen ssh
  bind :22022
  balance leastconn
  mode tcp
  server sshsrv1 172.16.100.6:22 check
  server sshsrv2 172.16.100.7:22 check		
```

#### bind
`bind [<address>]:<port_range> [, ...] [param*]`
- 作用: 定义前端服务监听的地址和端口

```
listen http_proxy
  bind :80,:443                       # 监听多个端口
  bind 10.0.0.1:10080,10.0.0.1:10443  
  bind /var/run/ssl-frontend.sock user root mode 600 accept-proxy # 监听本地 sock
```

#### default_backend
`default_backend <backend>`
- 作用: 设定默认的backend，用于frontend中

#### maxconn
`maxconn <conns>`:
- 作用: 为指定的frontend定义其最大并发连接数；默认为2000；
- 说明: 如果未设置，会继承 global 中的配置


## 2.2 后端主机配置
#### balance
`balance <algorithm> [ <arguments> ]`  
`balance url_param <param> [check_post]`
- 作用: 定义后端服务器组内的服务器调度算法
- `algorithm`: 算法
  - `roundrobin`:
    - 动态算法, 支持权重的运行时调整，支持慢启动
    - 每个后端中最多支持4095个server；
  - `static-rr`:
    - 静态算法，不支持权重的运行时调整及慢启动
    - 后端主机数量无上限；
  - `leastconn`:
    - 推荐使用在具有较长会话的场景中，例如MySQL、LDAP等；
  - `first`:
    - 根据服务器在列表中的位置，自上而下进行调度；
    - 前面服务器的连接数达到上限，新请求才会分配给下一台服务；
  - `source`:
    - 源地址hash，hash 算法下面的 `hash-type` 选项指定
    - `hash-type map-based`:除权取余法，这种方法不支持权重的运行时调整，也不支持慢启动，但资源耗费少
    - `hash-type consistent`: 一致性哈希，支持权重运行时调整，也支持慢启动，但是资源耗费多
  - `uri`:
    - 目标地址 hash，对URI的左半部分做hash计算
    - hash 算法可由 `hash-type` 指定，默认是 `map-based`
    - 完整 url: `<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>``
    - 左半部分: `/<path>;<params>`
    - 整个uri: `/<path>;<params>?<query>#<frag>`
  - `url_param`:
    - 对用户请求的uri的 `<params>`中的部分的参数的值作hash计算
    - hash 算法可由 `hash-type` 指定，默认是 `map-based`
    - 通常用于追踪用户，以确保来自同一个用户的请求始终发往同一个Backend Server
  - `hdr(<name>)`:
    - 对于每个http请求，此处由`<name>`指定的http首部将会被取出做hash计算
    - hash 算法可由 `hash-type` 指定，默认是 `map-based`
    - 没有有效值的会被轮询调度；
    - eg:  `hdr(Cookie)`

#### hash-type
`hash-type <method> <function> <modifier>`
- 作用: 指定 hash 计算的算法
- `method`:
  - `map-based`: 除权取余法，哈希数据结构是静态的数组；
  - `consistent`: 一致性哈希，哈希数据结构是一个树；
- `function`: 哈希函数，eg: `sdbm, djb2, wt6`



#### default-server
`default-server [param*]`
- 作用: 为backend中的各server设定默认选项
- 选项: 同 serve

#### server
`server <name> <address>[:[port]] [param*]`
- 作用: 定义后端主机的各服务器及其选项；
- `name`: 服务器在haproxy上的内部名称；出现在日志及警告信息
- `address`: 服务器地址，支持使用主机名；
- `[:[port]]`: 端口映射；省略时，表示同bind中绑定的端口；
- `[param*]`: 参数
  - `maxconn <maxconn>`: 当前server的最大并发连接数；
  - `backlog <backlog>`: 当前server的连接数达到上限后的后援队列长度；
  - `backup`: 设定当前server为备用服务器；
  - `check`: 对当前server做健康状态检测；
    - `addr` : 检测时使用的IP地址；
    - `port` : 针对此端口进行检测；
    - `inter <delay>`: 连续两次检测之间的时间间隔，默认为2000ms;
    - `rise <count>`: 连续多少次检测结果为“成功”才标记服务器为可用；默认为2；
    - `fall <count>`: 连续多少次检测结果为“失败”才标记服务器为不可用；默认为3；
    - 注意: httpchk，"smtpchk", "mysql-check", "pgsql-check" and "ssl-hello-chk" 用于定义应用层检测方法；
  - `cookie <value>`: 为当前server指定其cookie值，用于实现基于cookie的会话黏性；
  - `disabled`: 标记为不可用；
  - `redir <prefix>`: 将发往此server的所有GET和HEAD类的请求重定向至指定的URL；
  - `weight <weight>`: 权重，默认为1; 		

#### cookie
`cookie <name> options`
- 作用: 启用基于 cookie 的用户绑定
- 参数: `<name>` 待操作的 cookie 的键
- 选项:
  - `rewirte`: 重写
  - `insert`: 插入
  - `prefix`: 前缀
  - `nocache`: 只对非从缓存中响应的值进行操作
  - `indirect`: 如果对应的cookie 键已经存在值，则不修改直接发送给客户端
- 说明: cookie 的值有 `server cookie<value>` 配置

```
# 基于cookie的session sticky的实现:
backend websrvs
  cookie WEBSRV insert nocache indirect
  server srv1 172.16.100.6:80 weight 2 check rise 1 fall 2 maxconn 3000 cookie srv1
  server srv2 172.16.100.7:80 weight 1 check rise 1 fall 2 maxconn 3000 cookie srv2				
````

### 2.3 健康状态检测
#### option httpchk
`option httpchk <method> <uri> <version>`
- 作用: 对后端服务器做http协议的健康状态检测,基于http协议的7层健康状态检测机制；
- 参数:
  - `uri`: 指定检测的链接
  - `method`: 指定检测使用的请求方法
  - `version`: 指定发送的 http 的协议版本

```
backend https_relay
    mode tcp
    option httpchk OPTIONS * HTTP/1.1\r\nHost:\ www
    server apache1 192.168.1.1:443 check port 80
```

#### http-check expect
`http-check expect [!] <match> <pattern>`
- 作用: 指定健康状态检测的内容
- `[!]`: 表示取反操作
- `match`:					
  - `status`: 完全匹配响应码
  - `rstatus`: 以正则表达式匹配响应码
  - `string`: 完全匹配响应内容
  - `rstring`: 以正则表达式匹配响应内容
- `pattern`: 字符串或正则表达式，表示 `match` 指定的选项要匹配的内容

```

http-check expect status 200
http-check expect ! string SQL\ Error
http-check expect ! rstatus ^5

# check that we have a correct hexadecimal tag before /html
http-check expect rstring <!--tag:[0-9a-f]*</html>
```

### 2.4 统计接口
`stats enable`
- 作用: 启用统计页；基于默认的参数启用stats page；
- 默认启用参数:
  - `stats uri`   : /haproxy?stats
  - `stats realm` : "HAProxy Statistics"
  - `stats auth`  : no authentication
  - `stats scope` : no restriction

`stats auth <user>:<passwd>`
- 作用: 认证时的账号和密码，可使用多次；

`stats realm <realm>`
- 作用: 认证时的realm；

`stats uri <prefix>``
- 作用: 自定义stats page uri

`stats refresh <delay>`
- 作用: 设定自动刷新时间间隔；

`stats admin { if | unless } <cond>`
- 作用: 启用stats page中的管理功能

`stats hide-version`
- 作用: 隐藏页面的有关 haproxy 的版本信息

```
# 配置示例:
listen stats
  bind :9099
  stats enable
  stats realm HAPorxy\ Stats\ Page
  stats auth admin:admin
  stats admin if TRUE		
```

### 2.5 自定义错误页
#### errorfile
`errorfile <code> <file>`
- 作用: 自定义错误响应页，响应页的内容由文件指定
- `<code>`: HTTP 响应码， HAProxy 目前支持自定义 `200, 400, 403, 408, 500, 502, 503, and 504`
- `<file>`: 响应内容所在的位置

```
errorfile 400 /etc/haproxy/errorfiles/400badreq.http
errorfile 408 /dev/null  # workaround Chrome pre-connect bug
errorfile 403 /etc/haproxy/errorfiles/403forbid.http
errorfile 503 /etc/haproxy/errorfiles/503sorry.http
```

`errorloc <code> <url>`   
`errorloc302 <code> <url>`  
`errorloc303 <code> <url>`  
- 作用: 自定义错误响应页，错误页的内容由 url 指定
- 说明: `errorloc302/303` 会以 `302/303` 响应码响应客户端而不是原本的响应码

### 2.6 首部处理
#### option forwardfor
`option forwardfor [except <network>] [header <name>] [if-none]`
- 作用:
  - 用于向后端主发送真实的客户端IP
  - 在由haproxy发往后端主机的请求报文中添加`X-Forwarded-For`首部，其值为前端客户端的地址
- 选项:
  - `[except <network>]`: 请求报文来自此处指定的网络时不予添加此首部；
  - `[header <name> ]`: 使用自定义的首部名称，而非`X-Forwarded-For`
  - `[if-none]`: 只在不存在 `X-Forwarded-For` 时才添加此报文首部

#### reqadd/rspadd
`reqadd  <string> [{if | unless} <cond>]`
- 作用: 向发送给后端服务器的请求添加首部字段

`rspadd <string> [{if | unless} <cond>]`
- 作用: 向发送给客户端响应中添加首部字段
- eg: `rspadd X-Via: HAPorxy`

`reqdel  <search> [{if | unless} <cond>]`    
`reqidel <search> [{if | unless} <cond>]`
- 作用: 删除正则表达式匹配到的，发往后端服务器的请求的首部字段
- 参数: `search` 正则表达式，用于匹配首部字段及其值
- 说明: `reqidel` 表示在进行正则表达式匹配时不区分大小写

`rspdel  <search> [{if | unless} <cond>]`   
`rspidel <search> [{if | unless} <cond>]`
- 作用: 删除正则表达式匹配到的，发往客户端的的响应的的首部字段
- 参数: `search` 正则表达式，用于匹配首部字段及其值
- 说明: `rspidel` 表示在进行正则表达式匹配时不区分大小写

```
# remove X-Forwarded-For header and SERVER cookie
reqidel ^X-Forwarded-For:.*
reqidel ^Cookie:.*SERVER=

rspidel ^Server:.*
```

### 2.7 日志系统
#### log
`log global`  
- 作用: 从 global 全局配置段继承日志配置

`no log`
- 作用: 不记录日志

`log <address> [len <length>] <facility> [<level> [<minlevel>]]`
- 作用: 自定义日志记录
- 注意: 一个配置段内，log 日志记录只能使用两次，如果使用`log global`，并且global 配置了两次log，算作两次

```
# 注意：默认发往本机的日志服务器，配置如下
vim /etc/rsyslog.conf
local2.*      /var/log/local2.log
$ModLoad imudp
$UDPServerRun 514
```

#### log-format
`log-format <string>`
- 作用: 自定义日志记录格式

#### capture
`capture cookie <name> len <length>`
- 作用: 提取并记录请求和响应的 cookie 中特定字段的值
- 参数:
  - `<name>`: 指定的首部字段
  - `len <length>`: 最大记录的长度

`capture request header <name> len <length>`
- 作用: 提取并记录请求首部特定字段的值
- 参数:
  - `<name>`: 指定的首部字段
  - `len <length>`: 最大记录的长度

`capture response header <name> len <length>`
- 作用: 提取并记录响应首部特定字段的值
- 参数:
  - `<name>`: 指定的首部字段
  - `len <length>`: 最大记录的长度

```
capture request header Host len 15
capture request header X-Forwarded-For len 15
capture request header Referer len 15
```

### 2.8 压缩功能
#### compression algo
`compression algo <algorithm>`
- 作用：启用http协议的压缩机制，指明压缩算法gzip, deflate；

#### compression type
`compression type <mime type>`
- 作用: 指明压缩的MIMI类型；

```
compression algo gzip
compression type text/html text/plain
```

### 2.9 连接超时时长		
`timeout client <timeout>`
- 作用: 客户端非活动状态的超时时长

`timeout server <timeout>`
- 作用: 客户端与服务器端建立连接后，等待服务器端的超时时长，

`timeout http-keep-alive <timeout>`
- 作用: 持久连接的持久时长；

`timeout http-request <timeout>`
- 作用: 定义保持连接的超时时长

`timeout connect <timeout>`
- 作用: haproxy 连接后端主机的超时时长

`timeout client-fin <timeout>`
- 作用: 半关闭状态连接，client端非活动超时时间

`timeout server-fin <timeout>`
- 作用: 半关闭状态连接，server端非活动超时时间
