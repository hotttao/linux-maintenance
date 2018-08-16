# 20.4 httpd2.2 的基础配置
本节我们来讲解 httpd2.2 的基础配置

## 1. httpd-2.2 配置文件格式
```
$ grep "^###" /etc/httpd/conf/httpd.conf
### Section 1: Global Environment
### Section 2: 'Main' server configuration
### Section 3: Virtual Hosts

$ ll /etc/httpd/
总用量 0
drwxr-xr-x. 2 root root  37 5月  22 19:20 conf
drwxr-xr-x. 2 root root 151 5月   8 09:35 conf.d
lrwxrwxrwx. 1 root root  19 2月  10 2018 logs -> ../../var/log/httpd
lrwxrwxrwx. 1 root root  29 2月  10 2018 modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx. 1 root root  10 2月  10 2018 run -> /run/httpd
```
httpd2.2 的主配置文件是：`/etc/httpd/conf/httpd.conf`，其分成三个部分:
- Section 1: Global Environment  全局配置
- Section 2: 'Main' server configuration 主服务配置段
- Section 3: Virtual Hosts 虚拟主机配置段

配置格式为 `directive  value`, 比如`ServerRoot "/etc/httpd"`
- directive：配置参数，不区分字符大小写；
- value：参数值，为路径时，是否区分字符大小写，取决于文件系统；



## 2. httpd-2.2 常用配置
`httpd -t`，可用于检查配置文件是否有语法错误，常用的配置选项如下所示

### 2.1 修改监听的IP和PORT
`Listen  [IP:]PORT`
1. 省略IP表示为0.0.0.0；
2. Listen指令可重复出现多次，以监听多个IP地址和端口；
    - `Listen  80`
    - `Listen  8080`
3. 修改监听socket，重启服务进程方可生效；

### 2.2 持久连续
Persistent Connection：
- tcp连续建立后，每个资源获取完成后不全断开连接，而是继续等待其它资源请求的进行；
- 如何断开
    - 数量限制
    - 时间限制
- 副作用：
    - 对并发访问量较大的服务器，长连接机制会使得后续某些请求无法得到正常 响应；
    - 折衷：使用较短的持久连接时长，以及较少的请求数量；
    - httpd-2.4 支持毫秒级持久时间
```
# 配置
KeepAlive  On|Off
KeepAliveTimeout  15
MaxKeepAliveRequests  100

# 测试：
telnet  WEB_SERVER_IP  PORT
GET  /URL  HTTP/1.1
Host: WEB_SERVER_IP
```
### 2.3 MPM
- Multipath Process Module 多道处理模块
- 功能;
    - httpd-2.2不支持同时编译多个MPM模块，所以只能编译选定要使用的那个；
    - CentOS 6的rpm包为此专门提供了三个应用程序文件，httpd(prefork), httpd.worker, httpd.event，分别用于实现对不同的MPM机制的支持；
    - 确认现在使用的是哪下程序文件的方法：`ps  aux  | grep httpd`
    - 默认使用的为/usr/sbin/httpd，其为prefork的MPM模块
- 配置
    - 更换使用httpd程序，以支持其它MPM机制；
        - `/etc/sysconfig/httpd`, `HTTPD=/usr/sbin/httpd.{worker,event}`
    - 注意：重启服务进程方可生效

```
# 查看当前使用MPM模式
ps aux|grep httpd

# 查看httpd程序的模块列表：
## 查看静态编译的模块(先确定使用的MPM):
httpd  -l
httpd.event -l
httpd.worker -l

## 查看静态编译及动态编译的模块:
httpd  -M
httpd.envent -M
httpd.woker -M
```



#### prefork的配置
```        
# /etc/httpd/conf/httpd.conf
<IfModule prefork.c>
StartServers    8　　    ＃ 服务器启动时，启动的进程数
MinSpareServers 5        # 最小空闲进程数
MaxSpareServers  20
ServerLimit  256          # 为 MaxClients 提供的最大进程数
MaxClients    256        # 并发的最大客户端请求数
MaxRequestsPerChild  4000 # 一个进程能处理的请求总数，超过会自动销毁
</IfModule>        
```

#### worker的配置：
```
# /etc/httpd/conf/httpd.conf
<IfModule worker.c>
StartServers    4    # 服务器启动时启动的进程数
MaxClients      300
MinSpareThreads  25
MaxSpareThreads  75
ThreadsPerChild  25    # 每个进程启动的线程数
MaxRequestsPerChild  0
</IfModule>                        
```

PV，UV
PV：Page View
UV: User View

### 2.4 DSO
- 作用: 实现模块加载:  
- 配置指令：`LoadModule  mod_name  mod_path`
- 模块文件路径可使用相对路径：
  - 相对于`ServerRoot`（默认/etc/httpd）
  - `/etc/httpd/modules/`  --> `/usr/lib64/httpd/modules(动态模块位置)`

```
LoadModule access_compat_module modules/mod_access_compat.so
LoadModule actions_module modules/mod_actions.so
LoadModule alias_module modules/mod_alias.so

```

### 2.5 定义'Main' server的文档页面路径
- 配置: `DocumentRoot`
- 文档路径映射：DoucmentRoot指向的路径为URL路径的起始位置, 其相当于站点URL的根路径；

```
DocumentRoot "/var/www/html"
(FileSystem) /var/www/html/index.html    -->  (URL)/index.html
```

### 2.6 站点访问控制常见机制
- 作用: 是否允许用户访问站点资源
- 访问控制机制
    1. 基于来源地址
    2. 基于账号
- 资源指定方式
    1. 文件系统路径
    2. URL路径
- 访问控制机制的实现
    1. 基于来源地址，通过文件路径实现访问控制机制
    2. 基于来源地址，通过 url 实现访问控制机制
    3. 基于账号，过文件路径实现访问控制机制
    4. 基于账号，通过 url 实现访问控制机制

```
文件系统路径：
<Directory  "">
...
</Directory>

<File  "">
...
</File>

<FileMatch  "PATTERN">
...
</FileMatch>
```

```
URL路径：
<Location  "">
...
</Location>

<LocationMatch "">
...
</LocationMatch>

```

#### Directory 中“基于源地址”实现访问控制：
```
<Directory  "/var/www/html">
Options All，None，Indexes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
AllowOverride None
</Directory>

```
1. Options
    - 作用: 后跟1个或多个以空白字符分隔的“选项”列表；
    - 选项:
        - Indexes：指明的URL路径下不存在与定义的主页面资源相符的资源文件时，返回索引列表给用户； -- 危险，生产环境不能启用
        - FollowSymLinks：允许跟踪符号链接文件所指向的源文件；-- 危险
        - None：关闭所有选项
        - All：启动全部选项
2. AllowOverride
    - 作用:
        - 与访问控制相关的指令可以放在.htaccess文件，每个目录下都可以有一个；用于自定义每个目录的访问权限
        - 启用 .htaccess 会对性能产生重大影响，不建议启用
        - Directory 中的配置相当于全局配置
    - 选项:
        - All: Directory 中的全局配置，覆盖 htaccess 中的配置
        - None：Directory 中的全局配置，不会覆盖 htaccess 中的配置
3. order和allow、deny
    - 作用: 基于来源地址的访问控制
    - order：定义生效次序；写在后面的表示默认法则；
        - Order allow，deny: 白名单
        - Order deny，allow: 黑名单
    - Allow from IP, Deny from IP: 注明来源IP
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
    Require all granted
    Order allow,deny
    Allow from 192.168.1
    # Deny from 192.168.1.104
</Directory>
```

### 2.6 定义站点主页面：
- `DirectoryIndex  index.html  index.html.var`
- 找不到主页面时，httpd 会通过 /etc/httpd/conf.d/weibocom.conf 提供默认主页面

### 2.7 定义路径别名
- 格式：Alias  /URL/  "/PATH/TO/SOMEDIR/"
- eg:
    ```
    DocumentRoot "/www/htdocs"
    http://www.magedu.com/download/bash-4.4.2-3.el6.x86_64.rpm
    /www/htdocs/download/bash-4.4.2-3.el6.x86_64.rpm

    Alias  /download/  "/rpms/pub/"
    http://www.magedu.com/download/bash-4.4.2-3.el6.x86_64.rpm
    /rpms/pub/bash-4.4.2-3.el6.x86_64.rpm

    http://www.magedu.com/images/logo.png
    /www/htdocs/images/logo.png
    ```

### 2.8 设定默认字符集
- AddDefaultCharset  UTF-8

### 2.9 日志设定
```
# 错误日志：
ErrorLog  logs/error_log
LogLevel  warn

# 访问日志：
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %b" common

CustomLog  logs/access_log  combined # combined 为 LogFormat 名
```

- 日志类型：访问日志 和 错误日志
- LogLevel 可选值：debug, info, notice, warn, error, crit, alert, emerg)
- LogFormat format strings:
    - http://httpd.apache.org/docs/2.2/mod/mod_log_config.html#formats
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


### 2.10 基于用户的访问控制
- 认证质询：
    - WWW-Authenticate：响应码为401，拒绝客户端请求，并说明要求客户端提供账号和密码；
- 认证：
    - Authorization：客户端用户填入账号和密码后再次发送请求报文；认证通过时，则服务器发送响应的资源；
- 认证方式有两种：
      - basic：明文
      - digest：消息摘要认证
- 安全域：需要用户认证后方能访问的路径；应该通过名称对其进行标识，以便于告知用户认证的原因；

#### 用户的账号和密码存放于何处
- 虚拟账号：仅用于访问某服务时用到的认证标识
- 存储：
      - 文本文件；
      - SQL数据库；
      - ldap目录存储；

#### basic认证配置示例：
```
# 1.  定义安全域
<Directory "">
Options None
AllowOverride None
AuthType Basic
AuthName "String“
AuthUserFile  "/PATH/TO/HTTPD_USER_PASSWD_FILE"
Require  user  username1  username2 ...
# 允许账号文件中的所有用户登录访问：
# Require  valid-user
</Directory>

# 2. 提供账号和密码存储（文本文件）
# 使用专用命令完成此类文件的创建及用户管理
> htpasswd  [options]  /PATH/TO/HTTPD_PASSWD_FILE  username
    # -c：自动创建此处指定的文件，因此，仅应该在此文件不存在时使用；
    # -m：md5格式加密
    # -s: sha格式加密
    # -D：删除指定用户
```

#### 另外：基于组账号进行认证；
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
> htpasswd -m /etc/httpd/conf.d/.http_passwd  tom
> vim /etc/httpd/conf.d/.http_group
  GRP_NAME: tom admin
```

### 2.11 虚拟主机
- 站点标识： socket
- IP相同，但端口不同；
- IP不同，但端口均为默认端口；
- FQDN不同；
      - 请求报文中首部
      - Host: www.magedu.com
- 有三种实现方案：
    - 基于ip：    为每个虚拟主机准备至少一个ip地址；
    - 基于port：为每个虚拟主机使用至少一个独立的port；
    - 基于FQDN:  为每个虚拟主机使用至少一个FQDN；
    - 基于hostname: 为每个虚拟主机准备至少一个专用的 hostname， 常用
    - 注意：一般虚拟机不要与中心主机混用；因此，要使用虚拟主机，得先禁用'main'主机；
    - 禁用方法：`注释中心主机的DocumentRoot指令即可`；

#### 虚拟主机的配置方法：
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

#### 基于IP的虚拟主机示例：
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

####  基于端口的虚拟主机：
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

#### 基于 hostname 的虚拟主机：
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

### 2.12 status页面
LoadModule  status_module  modules/mod_status.so
```
<Location /server-status>
    SetHandler server-status
    Order allow,deny
    Allow from 172.16
</Location>
```
