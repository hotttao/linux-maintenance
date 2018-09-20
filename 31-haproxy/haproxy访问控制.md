# 32.3 haproxy 访问控制
本节我们来 介绍 haproxy 配置的第二部分，acl 访问控制。

## 1. acl
acl 能分类请求和响应报文，进而根据分类结果作出转发决策和访问控制。

### 1.1 acl 使用
`acl <aclname> <criterion> [flags] [operator] [<value>]`
- 作用: 根据设置的条件分类请求和响应报文
- 参数:
  - `<aclname>`:
    - 作用: acl 的名称标识，以便在后续的访问控制和转发中进行引用
    - 命名: 可用字符包括大小写子母，数字 `-`，`_`，`.`， `:`，大小写敏感的
    - 说明: 多个 acl 可以使用同一个名字，彼此之间属于 OR 关系
  - `<criterion>`: 匹配的标准，表示要检查什么内容
  - `[operator]`: 匹配操作，比如等于，小于，或正则表达式匹配
  - `<value>`: 标准匹配的目标值
  - `[flags]`: 比较控制标识
- 过程: acl 的匹配过程就是把`<criterion>`指定的匹配标准和 `<value>`指定的值基于`[operator]`指定的操作符作比较操作，如果符合条件则为真，否则为假

#### value
value 值可以是以下类型:
- boolean
- integer or integer range
- IP address / network
- string，字符串的比较分为如下多种类型
  - exact: criterion表示的值 与 value 完全匹配  
  - substring: value 是criterion表示值的子串
  - suffix: value 是criterion表示值的后缀
  - prefix: value 是criterion表示值的前串
  - subdir: value 是criterion表示路经中的子路经
  - domain: value 是criterion表示域名的的子域
- regular expression
- hex block

#### flags
flags 有如下几个选项
- `-i` : 比较时忽略大小写.
- `-m` : 使用一个特殊的模式匹配方法，很少用到
- `-n` : 禁止 DNS 反向解析
- `-u` : 禁止两个 acl 使用相同的名称
- `--` : flag的结束符标记，强制结束 flag

#### operator
比较操作有如下几种类型
- 匹配整数值：eq、ge、gt、le、lt
- 匹配字符串：
  - exact match     (-m str) : 精确匹配
  - substring match (-m sub) : 字串匹配
  - prefix match    (-m beg) : 前缀匹配
  - suffix match    (-m end) : 后缀匹配
  - subdir match    (-m dir) : 子路经匹配，即是否是`/`分隔的子串
  - domain match    (-m dom) : 域名匹配，即是否是`.`分隔的子串

#### acl作为条件时的逻辑关系
- AND: 默认多个条件使用空格分隔即表示逻辑与
- OR: 使用 `or` 或 `||` 表示逻辑与
- Not: 使用 `!` 表示取反

```
if invalid_src invalid_port
if invalid_src || invalid_port
if ! invalid_src invalid_port
```

#### criterion
criterion 用于指定匹配请求报文或响应报文的哪些内容
1. 匹配传输层和网络层报文中的内容
  - dst: 目标 ip
  - dst_port: 目标端口 integer
  - src: 源ip
  - src_port: 源端口
  - eg: `acl invalid_src  src  172.16.200.2`
2. 匹配 url 中的路经，即path 部分(`/path;<params>`)，string
  - path     : exact string match
  - path_beg : prefix match
  - path_dir : subdir match
  - path_dom : domain match
  - path_end : suffix match
  - path_len : length match
  - path_reg : regex match
  - path_sub : substring match
3. 匹配整个 url ，string
  - url     : exact string match
  - url_beg : prefix match
  - url_dir : subdir match
  - url_dom : domain match
  - url_end : suffix match
  - url_len : length match
  - url_reg : regex match
  - url_sub : substring match
3. 匹配请求报文首部中的特定字段，相同报文只会匹配最后一次出现，string
  - hdr([<name>[,<occ>]])     : exact string match
  - hdr_beg([<name>[,<occ>]]) : prefix match
  - hdr_dir([<name>[,<occ>]]) : subdir match
  - hdr_dom([<name>[,<occ>]]) : domain match
  - hdr_end([<name>[,<occ>]]) : suffix match
  - hdr_len([<name>[,<occ>]]) : length match
  - hdr_reg([<name>[,<occ>]]) : regex match
  - hdr_sub([<name>[,<occ>]]) : substring match					
4. 匹配响应状态码 `status`，integer
  - status

```
# 示例：
acl bad_curl hdr_sub(User-Agent) -i curl
block if bad_curl			
```

#### 预订义的 ACL
```
Pre-defined ACLs
ACL name	      Equivalent to	Usage
FALSE	          always_false	never match
HTTP	          req_proto_http	match if protocol is valid HTTP
HTTP_1.0	      req_ver 1.0	match HTTP version 1.0
HTTP_1.1	      req_ver 1.1	match HTTP version 1.1
HTTP_CONTENT	  hdr_val(content-length) gt 0	match an existing content-length
HTTP_URL_ABS	  url_reg ^[^/:]*://	match absolute URL with scheme
HTTP_URL_SLASH	url_beg /	match URL beginning with "/"
HTTP_URL_STAR	  url *	match URL equal to "*"
LOCALHOST	src   127.0.0.1/8	match connection from local host
METH_CONNECT	  method CONNECT	match HTTP CONNECT method
METH_GET	      method GET HEAD	match HTTP GET or HEAD method
METH_HEAD	      method HEAD	match HTTP HEAD method
METH_OPTIONS	  method OPTIONS	match HTTP OPTIONS method
METH_POST	      method POST	match HTTP POST method
METH_TRACE	    method TRACE	match HTTP TRACE method
RDP_COOKIE	   req_rdp_cookie_cnt gt 0	match presence of an RDP cookie
REQ_CONTENT	   req_len gt 0	match data in the request buffer
TRUE	         always_true	always match
WAIT_END	     wait_end	wait for end of content analysis		
```


## 2. 访问控制指令
#### use_backend
`use_backend <backend> [{if | unless} <condition>]`
- 作用: 当符合指定的条件时使用特定的backend；

#### block
`block { if | unless } <condition>`
- 作用: 当符合指定的条件时阻止七层 http 的访问


```
acl invalid_src src 172.16.200.2
block if invalid_src
errorfile 403 /etc/fstab
```

#### http-request
`http-request { allow | deny } [ { if | unless } <condition> ]`
- 作用: 控制七层的访问请求

#### tcp-request
`tcp-request connection {accept|reject}  [{if | unless} <condition>]`
- 作用: 四层的连接请求控制

```
listen ssh
  bind :22022
  balance leastconn
  acl invalid_src src 172.16.200.2
  tcp-request connection reject if invalid_src
  mode tcp
  server sshsrv1 172.16.100.6:22 check
  server sshsrv2 172.16.100.7:22 check backup			
```


## 3. 基于ACL的动静分离示例
需要注意的 haproxy 不能作为 fastcgi 的客户端，因此其后端主机不能是 phpfpm 这种 fastcgi 的应用程序服务器。后端服务器只能是 httpd 或者 nginx，并通过它们来反代 fpm。

```
frontend  web *:80
  acl url_static       path_beg       -i  /static /images /javascript /stylesheets
  acl url_static       path_end       -i  .jpg .gif .png .css .js .html .txt .htm

  use_backend staticsrvs          if url_static
  default_backend                 appsrvs

backend staticsrvs
  balance     roundrobin
  server      stcsrv1 172.16.100.6:80 check

backend appsrvs
  balance     roundrobin
  server  app1 172.16.100.7:80 check
  server  app1 172.16.100.7:8080 check

listen stats
  bind :9091
  stats enable
  stats auth admin:admin
  stats admin if TRUE		
```

## 4. 配置HAProxy支持https协议：
```
# 1 支持ssl会话
bind *:443 ssl crt /PATH/TO/SOME_PEM_FILE

# crt后的证书文件要求PEM格式，且同时包含证书和与之匹配的所有私钥；
cat  demo.crt demo.key > demo.pem

# 2 把80端口的请求重向定443；
bind *:80
redirect scheme https if !{ ssl_fc }

# 3 如何向后端传递用户请求的协议和端口
http_request set-header X-Forwarded-Port %[dst_port]
http_request add-header X-Forwared-Proto https if { ssl_fc }
```
