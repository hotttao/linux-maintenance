# 21.1 http协议高级进阶



## 1 http协议进阶
- 特性: stateless: 服务器无法持续追踪访问者来源
- 协议查看或分析的工具：tcpdump, tshark, wireshark

## 1.1 URL：
URL: Unifrom Resource Locator
- 基本语法：`<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>`
    - params: 参数 http://www.magedu.com/bbs/hello;gender=f
    - query： http://www.magedu.com/bbs/item.php?username=tom&title=abc
    - frag：https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html-single/

### 1.2 报文语法格式：
request报文
```
<method> <request-URL> <version>
<headers>

<entity-body>
```

response报文
```
<version> <status> <reason-phrase>
<headers>

<entity-body>
```
- method: 请求方法，标明客户端希望服务器对资源执行的动作 -- GET、HEAD、POST
- version: HTTP/<major>.<minor>
- status: 三位数字，如200，301, 302, 404, 502; 标记请求处理过程中发生的情况；
- reason-phrase：状态码所标记的状态的简要描述；
- headers：每个请求或响应报文可包含任意个首部；每个首部都有首部名称，后面跟一个冒号，而后跟上一个可选空格，接着是一个值；
- entity-body：请求时附加的数据或响应时附加的数据；


### 1.3 method(方法)：
- GET：从服务器获取一个资源；
- HEAD：只从服务器获取文档的响应首部；
- POST：向服务器发送要处理的数据；
- PUT：将请求的主体部分存储在服务器上；
- DELETE：请求删除服务器上指定的文档；
- TRACE：追踪请求到达服务器中间经过的代理服务器；
- OPTIONS：请求服务器返回对指定资源支持使用的请求方法；


### 1.4 status(状态码)：
- 1xx：100-101, 信息提示；
- 2xx：200-206, 成功
- 3xx：300-305, 重定向
- 4xx：400-415, 错误类信息，客户端错误
- 5xx：500-505, 错误类信息，服务器端错误

常用的状态码：
- 200： 成功，请求的所有数据通过响应报文的entity-body部分发送；OK
- 301： 永久重定向，请求的URL指向的资源已经被删除；但在响应报文中通过首部Location指明了资源现在所处的新位置；Moved Permanently
- 302： 临时重定向，与301相似，但在响应报文中通过Location指明资源现在所处临时新位置; Found
- 304： 条件式请求，客户端发出了条件式请求，但服务器上的资源未曾发生改变，则通过响应此响应状态码通知客户端；Not Modified
- 401： 需要输入账号和密码认证方能访问资源；Unauthorized
- 403： 请求被禁止；Forbidden
- 404： 服务器无法找到客户端请求的资源；Not Found
- 500： 服务器内部错误；Internal Server Error
- 502： 代理服务器从后端服务器收到了一条伪响应；Bad Gateway

### 1.5 headers：
- 格式：Name: Value
- 首部的分类：
    - 通用首部
    - 请求首部
    - 响应首部
    - 实体首部
    - 扩展首部
```
Cache-Control:public, max-age=600
Connection:keep-alive
Content-Type:image/png
Date:Tue, 28 Apr 2015 01:43:54 GMT
ETag:"5af34e-ce6-504ea605b2e40"
Last-Modified:Wed, 08 Oct 2014 14:46:09 GMT


Accept:image/webp,*/*;q=0.8
Accept-Encoding:gzip, deflate, sdch
Accept-Language:zh-CN,zh;q=0.8
Cache-Control:max-age=0
Connection:keep-alive
Host:access.redhat.com
If-Modified-Since:Wed, 08 Oct 2014 14:46:09 GMT
If-None-Match:"5af34e-ce6-504ea605b2e40"
Referer:https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html-single/Installation_Guide/index.html
User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.101 Safari/537.36
```


#### 通用首部：
- Date: 报文的创建时间
- Connection：连接状态，如keep-alive, close
