

### 日志系统
### log
`log global`  
`log <address> [len <length>] <facility> [<level> [<minlevel>]]`
`no log`

```
# 注意：默认发往本机的日志服务器，配置如下
local2.*      /var/log/local2.log
$ModLoad imudp
$UDPServerRun 514
```

`log-format <string>`
课外实践：参考文档实现combined格式的记录

`capture cookie <name> len <length>`
Capture and log a cookie in the request and in the response.

`capture request header <name> len <length>`
Capture and log the last occurrence of the specified request header.
capture request header X-Forwarded-For len 15

`capture response header <name> len <length>`
Capture and log the last occurrence of the specified response header.
capture response header Content-length len 9
capture response header Location len 15			

### 压缩功能
#### compression algo
`compression algo <algorithm>`
- 作用：启用http协议的压缩机制，指明压缩算法gzip, deflate；

#### compression type
`compression type <mime type>`
- 作用：：指明压缩的MIMI类型；



### 连接超时时长		
`timeout client <timeout>`
Set the maximum inactivity time on the client side. 默认单位是毫秒;

`timeout server <timeout>`
Set the maximum inactivity time on the server side.

`timeout http-keep-alive <timeout>`
持久连接的持久时长；

`timeout http-request <timeout>`
Set the maximum allowed time to wait for a complete HTTP request

`timeout connect <timeout>`
Set the maximum time to wait for a connection attempt to a server to succeed.

`timeout client-fin <timeout>`
Set the inactivity timeout on the client side for half-closed connections.

`timeout server-fin <timeout>`
Set the inactivity timeout on the server side for half-closed connections.


### 访问控制
`use_backend <backend> [{if | unless} <condition>]`
当符合指定的条件时使用特定的backend；

`block { if | unless } <condition>`
Block a layer 7 request if/unless a condition is matched

```
acl invalid_src src 172.16.200.2
block if invalid_src
errorfile 403 /etc/fstab
```

`http-request { allow | deny } [ { if | unless } <condition> ]`
Access control for Layer 7 requests

`tcp-request connection {accept|reject}  [{if | unless} <condition>]`
Perform an action on an incoming connection depending on a layer 4 condition

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

## acl
The use of Access Control Lists (ACL) provides a flexible solution to perform content switching and generally to take decisions based on content extracted from the request, the response or any environmental status.

`acl <aclname> <criterion> [flags] [operator] [<value>]`
- <aclname>：可用字符包括大小写子母，数字 `-`，`_`，`.`， `:`，大小写敏感的
- `<value>`的类型：
  - boolean
  - integer or integer range
  - IP address / network
  - string (exact, substring, suffix, prefix, subdir, domain)
  - regular expression
  - hex block
- `<flags>`
  - `-i` : ignore case during matching of all subsequent patterns.
  - `-m` : use a specific pattern matching method
  - `-n` : forbid the DNS resolutions
  - `-u` : force the unique id of the ACL
  - `--` : force end of flags. Useful when a string looks like one of the flags.
- `[operator]`
  - 匹配整数值：eq、ge、gt、le、lt
  - 匹配字符串：
    - exact match     (-m str) : the extracted string must exactly match the patterns ;
    - substring match (-m sub) : the patterns are looked up inside the extracted string, and the ACL matches if any of them is found inside ;
    - prefix match    (-m beg) : the patterns are compared with the beginning of the extracted string, and the ACL matches if any of them matches.
    - suffix match    (-m end) : the patterns are compared with the end of the extracted string, and the ACL matches if any of them matches.
    - subdir match    (-m dir) : the patterns are looked up inside the extracted string, delimited with slashes ("/"), and the ACL matches if any of them matches.
    - domain match    (-m dom) : the patterns are looked up inside the extracted string, delimited with dots ("."), and the ACL matches if any of them matches.


acl作为条件时的逻辑关系：
- AND (implicit)
- OR  (explicit with the "or" keyword or the "||" operator)
- Negation with the exclamation mark ("!")

  if invalid_src invalid_port
  if invalid_src || invalid_port
  if ! invalid_src invalid_port

<criterion> ：
dst : ip
dst_port : integer
src : ip
src_port : integer

  acl invalid_src  src  172.16.200.2

path : string
  This extracts the request's URL path, which starts at the first slash and ends before the question mark (without the host part).
    /path;<params>

  path     : exact string match
  path_beg : prefix match
  path_dir : subdir match
  path_dom : domain match
  path_end : suffix match
  path_len : length match
  path_reg : regex match
  path_sub : substring match

url : string
  This extracts the request's URL as presented in the request. A typical use is with prefetch-capable caches, and with portals which need to aggregate multiple information from databases and keep them in caches.

  url     : exact string match
  url_beg : prefix match
  url_dir : subdir match
  url_dom : domain match
  url_end : suffix match
  url_len : length match
  url_reg : regex match
  url_sub : substring match

req.hdr([<name>[,<occ>]]) : string
  This extracts the last occurrence of header <name> in an HTTP request.

  hdr([<name>[,<occ>]])     : exact string match
  hdr_beg([<name>[,<occ>]]) : prefix match
  hdr_dir([<name>[,<occ>]]) : subdir match
  hdr_dom([<name>[,<occ>]]) : domain match
  hdr_end([<name>[,<occ>]]) : suffix match
  hdr_len([<name>[,<occ>]]) : length match
  hdr_reg([<name>[,<occ>]]) : regex match
  hdr_sub([<name>[,<occ>]]) : substring match					

  示例：
    acl bad_curl hdr_sub(User-Agent) -i curl
    block if bad_curl					

status : integer
  Returns an integer containing the HTTP status code in the HTTP response.

```
Pre-defined ACLs
ACL name	Equivalent to	Usage
FALSE	always_false	never match
HTTP	req_proto_http	match if protocol is valid HTTP
HTTP_1.0	req_ver 1.0	match HTTP version 1.0
HTTP_1.1	req_ver 1.1	match HTTP version 1.1
HTTP_CONTENT	hdr_val(content-length) gt 0	match an existing content-length
HTTP_URL_ABS	url_reg ^[^/:]*://	match absolute URL with scheme
HTTP_URL_SLASH	url_beg /	match URL beginning with "/"
HTTP_URL_STAR	url *	match URL equal to "*"
LOCALHOST	src 127.0.0.1/8	match connection from local host
METH_CONNECT	method CONNECT	match HTTP CONNECT method
METH_GET	method GET HEAD	match HTTP GET or HEAD method
METH_HEAD	method HEAD	match HTTP HEAD method
METH_OPTIONS	method OPTIONS	match HTTP OPTIONS method
METH_POST	method POST	match HTTP POST method
METH_TRACE	method TRACE	match HTTP TRACE method
RDP_COOKIE	req_rdp_cookie_cnt gt 0	match presence of an RDP cookie
REQ_CONTENT	req_len gt 0	match data in the request buffer
TRUE	always_true	always match
WAIT_END	wait_end	wait for end of content analysis		
```

### 基于ACL的动静分离示例

```
frontend  web *:80
  acl url_static       path_beg       -i  /static /images /javascript /stylesheets
  acl url_static       path_end       -i  .jpg .gif .png .css .js .html .txt .htm

  use_backend staticsrvs          if url_static
  default_backend             appsrvs

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

### 配置HAProxy支持https协议：
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
