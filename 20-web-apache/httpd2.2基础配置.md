# 20.5 httpd2.2 的基础配置
本节我们来讲解 httpd2.2 的基础配置

## 1. httpd-2.2 配置文件格式
```
$ grep "^###" /etc/httpd/conf/httpd.conf
### Section 1: Global Environment
### Section 2: 'Main' server configuration
### Section 3: Virtual Hosts
```
httpd2.2 的主配置文件是：`/etc/httpd/conf/httpd.conf`，其分成三个部分:
- Section 1: Global Environment  全局配置
- Section 2: 'Main' server configuration 主服务配置段
- Section 3: Virtual Hosts 虚拟主机配置段

下面是一段配置示例
```
ServerRoot "/etc/httpd"
Include conf.modules.d/*.conf

<Directory />             # 目录访问权限配置
    AllowOverride none
    Require all denied
</Directory>

<IfModule dir_module>    # IfModule 判断动态模块是否存在，动态配置
    DirectoryIndex index.html
</IfModule>
```

配置的格式为 `directive  value`
- directive：配置参数，不区分字符大小写；
- value：参数值，为路径时，是否区分字符大小写，取决于文件系统；

## 2. httpd-2.2 常用配置
httpd2.2 的官方文档: http://httpd.apache.org/docs/2.2/

### 2.1 修改监听的IP和PORT
`Listen  [IP:]PORT`
1. 省略IP表示监听本地所有地址
2. Listen指令可重复出现多次，以监听多个IP地址和端口；
    - `Listen  80`
    - `Listen  8080`
3. 修改监听socket(不是新增)，需要重启服务进程才能生效；

### 2.2 持久连续
Persistent Connection
```
# 持久链接配置相关参数
KeepAlive  On|Off         # 是否启用持久连接
KeepAliveTimeout  15      # 持久链接最大连接时长
MaxKeepAliveRequests  100 # 持久链接最多处理的请求数
```
前面我们说过 tcp 连接有长连接短连接之分
- 长连接能降低请求响应的时间
- 但对并发访问量较大的服务器，长连接机制会使得后续某些请求无法得到正常 响应；
因此通常的折衷策略是，采用长连接，但是使用较短的持久连接时长，以及较少的请求数量。

http 与长连接相关的参数包括
1. `KeepAlive On|Off`: 是否启用持久连接
2. `KeepAliveTimeout time`: 持久链接最大连接时长,httpd-2.4 支持毫秒级持久时间
3. `MaxKeepAliveRequests`: 单个持久连接能够处理的对大请求数

### 2.3 MPM
MPM(Multipath Process Module) 多道处理模块,用来确定 httpd 响应用户请求的模型。
对于 httpd2.2:
- 不支持同时编译多个MPM模块，所以只能编译选定要使用的那个。
- CentOS 6的rpm包为此专门提供了三个应用程序文件，`httpd(prefork)`, `httpd.worker`, `httpd.event`，分别用于实现对不同的MPM机制的支持。
- 默认使用的为`/usr/sbin/httpd`，其为prefork的MPM模块

查看 httpd2.2 当前使用的 MPM 以及修改默认的 MPM 的方式如下:

```
# 1. 查看当前使用MPM模式
ps aux|grep httpd

# 2. 查看httpd程序的模块列表：
## 查看静态编译的模块(先确定使用的MPM):
httpd  -l
httpd.event -l
httpd.worker -l

## 查看静态编译及动态编译的模块:
httpd  -M
httpd.envent -M
httpd.woker -M

# 3. 更改 service 使用的 httpd 程序
vim /etc/sysconfig/httpd        
HTTPD=/usr/sbin/httpd.{worker,event}  # 修改 HTTPD 参数，重启服务进程方可生效
```

#### prefork的配置参数
```        
# /etc/httpd/conf/httpd.conf
<IfModule prefork.c>
  StartServers    8　　    ＃ 服务启动时，启动的进程数
  MinSpareServers 5        # 最小空闲进程数
  MaxSpareServers  20      # 最大空闲进程数
  ServerLimit  256          # 为 MaxClients 提供的最大进程数，通常等于 MaxClients
  MaxClients    256         # 并发的最大客户端请求数
  MaxRequestsPerChild  4000 # 一个进程能处理的请求总数，超过会自动销毁
</IfModule>        
```

#### worker的配置参数
```
# /etc/httpd/conf/httpd.conf
<IfModule worker.c>
StartServers    4      # 服务器启动时启动的进程数
MaxClients      300   
MinSpareThreads  25    # 最小的空闲线程数
MaxSpareThreads  75    # 最大的空闲线程数
ThreadsPerChild  25    # 每个进程启动的线程数
MaxRequestsPerChild  0 # 每个线成能处理的请求总数，超过会自动销毁，0 表示无限
</IfModule>                        
```

#### event 的配置
event 在 httpd2.2 中尚且属于测试阶段，不建议在线上使用

### 2.4 DSO
`LoadModule  mod_name  mod_path`
- 作用: 实现模块加载:  
- 参数:
  - `mod_name`: 模块的名称
  - `mod_path`: 模块的路经，可使用相对路径：相对于`ServerRoot`

```
LoadModule alias_module modules/mod_alias.so
#  modules/mod_alias.so -- > /etc/httpd/modules/mod_alias.so
# /etc/httpd/modules    ---> /usr/lib64/httpd/modules
```

### 2.5 'Main' server配置
`DocumentRoot Dir`
- 作用: 文档路径映射,DoucmentRoot指向的路径为URL路径的起始位置, 其相当于站点URL的根路径；

```
DocumentRoot "/var/www/html"
(FileSystem) /var/www/html/index.html    -->  (URL)/index.html
```

`ServerName www.example.com:80`
- 作用: 配置主服务的标识
- 默认:
  - 未设置此参数时， httpd 会自动反解监听的 IP 地址，反解失败，则默认为当前主机的主机名
  - 可以随意设置，如果没有注册 DNS 域名，也可以设置成 ip 地址

### 2.6 定义站点主页面
`DirectoryIndex  index.html  index.html.var`
- 作用: 配置当用户访问 URL 的指向是一个目录时，httpd 应该默认响应的内容
- 附注: 找不到主页面时，httpd 通过 `/etc/httpd/conf.d/weibocom.conf` 提供了一个默认主页面

### 2.7 定义路径别名
`Alias  /URL/  "/PATH/TO/SOMEDIR/"`
- 作用: 定义 url 访问资源的路经别名

```
DocumentRoot "/www/htdocs"
http://www.magedu.com/download/a.index --> /www/htdocs/download/a.index

Alias  /download/  "/rpms/pub/"
http://www.magedu.com/download/a.index  --> /rpms/pub/a.index

http://www.magedu.com/images/logo.png  ---> /www/htdocs/images/logo.png
```

### 2.8 设定默认字符集
`AddDefaultCharset  UTF-8`
- 作用: 设定默认字符集

### 2.9 日志设定
```
# 错误日志：
ErrorLog  logs/error_log     # 错误日志的路经
LogLevel  warn               # 日志的级别

# 定义日志格式
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %b" common

CustomLog  logs/access_log  combined   # 访问日志的路经，combined 为 LogFormat 名
```

`LogLevel level`
- 作用: 日志级别
- 可选值：debug, info, notice, warn, error, crit, alert, emerg

#### LogFormat 格式化字符串
http://httpd.apache.org/docs/2.2/mod/mod_log_config.html#formats

|标志宏|作用|
|:---|:---|
|%h|客户端IP地址|
|%l|Remote User, 通常为一个减号（“-”）|
|%u|Remote user (from auth; may be bogus if return status (%s) is 401)；非为登录访问时，其为一个减号|
|%t|服务器收到请求时的时间|
|%r|First line of request，即表示请求报文的首行；记录了此次请求的“方法”，“URL”以及协议版本|
|%>s|响应状态码|
|%b|响应报文的大小，单位是字节；不包括响应报文的http首部|
|%{Referer}i|请求报文中首部“referer”的值；即从哪个页面中的超链接跳转至当前页面的|
|%{User-Agent}i|请求报文中首部“User-Agent”的值；即发出请求的应用程序|


## 3. httpd2.2 访问控制
### 3.1 访问控制机制
访问控制值的时允许哪些用户访问哪些站点资源
1. 用户可以通过`来源地址` 或 `账号` 指定，
2. 资源可以通过`文件系统路径` 或 `URL` 指定

因此 httpd 的访问控制机制有如下四中实现方式:
1. 基于来源地址，通过文件路径实现访问控制机制
2. 基于来源地址，通过 url 实现访问控制机制
3. 基于账号，过文件路径实现访问控制机制
4. 基于账号，通过 url 实现访问控制机制

httpd2.2 与 httpd2.4 最大的不同之处在于，httpd2.4 如果未授权，默认是所有用户都无法访问的，而 http2.2 则默认允许访问。因此当在 httpd2.4 中添加 Alias 路经别名，或更改 DocumentRoot 时，必需配置相应的访问权限。

#### 资源界定
资源的文件系统路径由如下几种配置方式
```
<Directory  "">   # 目录
...
</Directory>

<File  "">        # 单文件
...
</File>

<FileMatch  "PATTERN">  # 文件路经匹配的正则表达式
...
</FileMatch>
```

URL 有如下两种配置方式
```
<Location  "">   # URL 地址
...
</Location>

<LocationMatch ""> # URL 匹配的正则表达式
...
</LocationMatch>
```

### 3.1 Directory 中“基于源地址”实现访问控制：
```
<Directory  "/var/www/html">
Options All，None，Indexes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
AllowOverride None
</Directory>

```
http2.2 访问控制参数如下:
1. `Options`
    - 作用: 后跟1个或多个以空白字符分隔的“选项”列表；
    - 选项:
        - `Indexes`：指明的URL路径下不存在与定义的主页面资源相符的资源文件时，返回索引列表给用户； -- 危险，生产环境不能启用
        - `FollowSymLinks`：允许跟踪符号链接文件所指向的源文件；-- 危险
        - `None`：关闭所有选项
        - `All`：启动全部选项
    - 说明: 选项前加上`-`，表示关闭，但是只能要么都有`-`，要么都没有
2. `AllowOverride`
    - 作用:
        - 与访问控制相关的指令可以放在.htaccess文件，每个目录下都可以有一个；用于自定义每个目录的访问权限
        - 启用 `.htaccess` 会对性能产生重大影响，不建议启用
        - `Directory` 中的配置相当于全局配置
    - 选项:
        - `All`: Directory 中的全局配置，覆盖 htaccess 中的配置
        - `None`：Directory 中的全局配置，不会覆盖 htaccess 中的配置
3. `order`和`allow`、`deny`
    - 作用: 基于来源地址的访问控制
    - `order`：定义生效次序；写在后面的表示默认法则；
        - `Order allow，deny`: 白名单
        - `Order deny，allow`: 黑名单
    - `Allow from IP`, `Deny from IP`: 注明来源IP
        - 来源地址：
            - IP
            - NetAddr: 子网
                - 172.16
                - 172.16.0.0
                - 172.16.0.0/16
                - 172.16.0.0/255.255.0.0

```
<Directory "/www/htdoc">
    AllowOverride None
    Options Indexes FollowSymLinks
    Order allow,deny
    Allow from 192.168.1
    # Deny from 192.168.1.104
</Directory>
```

### 3.2 基于用户的访问控制
WWW-Authenticate（认证质询）是 http 协议早期提供的用户认证机制。当用户请求受控资源时，服务器响应 401，拒绝客户端请求，并说明要求客户端提供账号和密码；浏览器接收到响应时，会弹出认证窗口，客户端用户填入账号和密码后再次发送请求报文；认证通过时，则服务器发送响应的资源。需要用户认证后方能访问的路径；应该通过名称对其进行标识，以便于告知用户认证的原因。

WWW-Authenticate 认证方式有两种：
- basic认证：会明文传输帐号和密码，不安全
- digest认证：

用户的账号和密码可存储在文本文件，SQL数据库，ldap目录存储。但是通常都是存在本地的文本文件中，因为 httpd 访问 mysql 的模块是非标准模块，需要单独编译安装。现在 web 站点基本都是通过表单进行身份认证，认证质询因为安全性和便利性的问题，其实很少使用。

#### basic认证配置示例
```
# 1.  定义安全域
<Directory "">
    Options None
    AllowOverride None
    AuthType Basic
    AuthName "String“                                # 认证提时字符串
    AuthUserFile  "/PATH/TO/HTTPD_USER_PASSWD_FILE"  # 帐号密码文件路经
    Require  user  username1  username2 ...          # 允许访问的用户  
    # Require  valid-user                    # 允许账号文件中的所有用户登录访问
</Directory>

# 2. 使用 htpassword 命令创建帐号密码文件
$ htpassword -cb /etc/httpd/.httpd tao tao
```

#### 基于组账号进行认证
```
# 1. 定义安全域
<Directory "">
    Options None
    AllowOverride None
    AuthType Basic
    AuthName "String“
    AuthUserFile  "/PATH/TO/HTTPD_USER_PASSWD_FILE"
    AuthGroupFile "/PATH/TO/HTTPD_GROUP_FILE"
    Require  group  grpname1  grpname2 ...
</Directory>

# 2. 创建用户账号和组账号文件；
  # 组文件：每一行定义一个组
  # GRP_NAME: username1  username2  ...

> htpasswd -m /etc/httpd/conf.d/.http_passwd  tom  # 创建帐号文件
> vim /etc/httpd/conf.d/.http_group                # 创建组文件
  admin: tom jerry                            # tom, jerry 为 admin 组
```

## 4. 虚拟主机
通常如果我们在同一主机上提供了多个彼此毫不相干的服务时，通常是将他们隔离开，放在不同的域名下进行管理，而不是放在同一域名下；以免某一站点被劫持，所有站点都被劫持。虚拟主机就是帮助我们实现一个物理服务器服务多个网站的功能。

web 服务通过 tcp 进行通信，即每个服务都监听在一个特定的套接子socket 上，`socket = ip + 端口`，所以实现虚拟主机就有如下几种方法:
1. IP相同，但端口不同；
2. IP不同，但端口均为默认端口；
3. FQDN不同: 为不同的 web 服务配置不同的域名

需要注意的是请求报文首部中的 Host 字段会保留浏览器中用户请求的域名，因此服务器端可以通过解析Host 字段来路由请求。httpd 中心主机与虚拟主机不能混用，httpd2.2 中使用虚拟主机必需禁用'main'主机,禁用方法：`注释中心主机的DocumentRoot指令即可`。http2.4 中启用虚拟主机后，中心主机会自动禁用。

### 4.1 虚拟主机的配置方法
所有能用在中心主机的配置，均可以用在虚拟主机中

```
<VirtualHost  IP:PORT>
    ServerName FQDN  
    DocumentRoot  ""

    # 其它可用指令：
    ServerAlias：虚拟主机的别名；可多次使用；
    ErrorLog：
    CustomLog：
    <Directory "">
    ...
    </Directory>
    Alias
    ...

</VirtualHost>
```

### 4.2 基于IP的虚拟主机示例：
```
<VirtualHost 192.168.1.120:80>
    ServerName web1.tao.com
    DocumentRoot "/vhosts/web1/htdocs"
    <Directory "/vhosts/web1/htdocs">
        Options None
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost 172.16.100.7:80>
  ServerName www.b.net
  DocumentRoot "/www/b.net/htdocs"
</VirtualHost>

<VirtualHost 172.16.100.8:80>
  ServerName www.c.org
  DocumentRoot "/www/c.org/htdocs"
</VirtualHost>
```

### 4.3  基于端口的虚拟主机：
```
<VirtualHost 172.16.100.6:80>
  ServerName www.a.com
  DocumentRoot "/www/a.com/htdocs"
</VirtualHost>

<VirtualHost 172.16.100.6:808>
  ServerName www.b.net
  DocumentRoot "/www/b.net/htdocs"
</VirtualHost>

<VirtualHost 172.16.100.6:8080>
  ServerName www.c.org
  DocumentRoot "/www/c.org/htdocs"
</VirtualHost>
```

### 4.4 基于 hostname 的虚拟主机：
```
# 重要，http2.2 中需要指明 NameVirtualHost
NameVirtualHost 172.16.100.6:80

<VirtualHost 172.16.100.6:80>
  ServerName www.a.com
  DocumentRoot "/www/a.com/htdocs"
</VirtualHost>

<VirtualHost 172.16.100.6:80>
  ServerName www.b.net
  DocumentRoot "/www/b.net/htdocs"
</VirtualHost>

<VirtualHost 172.16.100.6:80>
  ServerName www.c.org
  DocumentRoot "/www/c.org/htdocs"
</VirtualHost>                                            
```

## 5. status页面
```
LoadModule  status_module  modules/mod_status.so
<Location /server-status>
    SetHandler server-status
    Order allow,deny
    Allow from 172.16
</Location>
```
