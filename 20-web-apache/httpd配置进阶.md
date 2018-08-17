# 20.6 httpd 配置进阶
## 3. httpd-2.2的常见配置(2)
### 3.2 user/group
- 指定以哪个用户的身份运行httpd服务进程；
    - User apache
    - Group apache
    - SUexec

### 3.3 使用mod_deflate模块压缩页面优化传输速度
- 适用场景：
    - 节约带宽，额外消耗CPU；同时，可能有些较老浏览器不支持；
    - 压缩适于压缩的资源，例如文件文件；
```
SetOutputFilter DEFLATE

# mod_deflate configuration
# Restrict compression to these MIME types
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE text/css

# Level of compression (Highest 9 - Lowest 1)
DeflateCompressionLevel 9

# Netscape 4.x has some problems.
BrowserMatch ^Mozilla/4  gzip-only-text/html

# Netscape 4.06-4.08 have some more problems
BrowserMatch  ^Mozilla/4\.0[678]  no-gzip

# MSIE masquerades as Netscape, but it is fine
BrowserMatch \bMSI[E]  !no-gzip !gzip-only-text/html
```

### 3.4 配置httpd支持https



#### SSL会话的简化过程
1. 客户端发送可供选择的加密方式，并向服务器请求证书；
2. 服务器端发送证书以及选定的加密方式给客户端；
3. 客户端取得证书并进行证书验正：
    - 验正证书来源的合法性；用CA的公钥解密证书上数字签名；
    - 验正证书的内容的合法性：完整性验正
    - 检查证书的有效期限；
    - 检查证书是否被吊销；
    - 证书中拥有者的名字，与访问的目标主机要一致；
4. 客户端生成临时会话密钥（对称密钥），并使用服务器端的公钥加密此数据发送给服务器，完成密钥交换；
5. 服务用此密钥加密用户请求的资源，响应给客户端；
6. 注意：SSL会话是基于IP地址创建；所以单IP的主机上，仅可以使用一个https虚拟主机；

#### 配置httpd支持https：
1. 为服务器申请数字证书；
    - 测试：通过私建CA发证书
        - 创建私有CA
        - 在服务器创建证书签署请求
        - CA签证
2. 配置httpd支持使用ssl，及使用的证书；
    - yum -y install mod_ssl
    - 配置文件：/etc/httpd/conf.d/ssl.conf
        - DocumentRoot
        - ServerName
        - SSLCertificateFile
        - SSLCertificateKeyFile
3. 测试基于https访问相应的主机；
    - openssl  s_client  [-connect host:port] [-cert filename] [-CApath directory] [-CAfile filename]

## 4. httpd自带的工具程序
1. htpasswd：basic认证基于文件实现时，用到的账号密码文件生成工具；
2. apachectl：httpd自带的服务控制脚本，支持start和stop,restart；
3. apxs：由httpd-devel包提供，扩展httpd使用第三方模块的工具；
4. rotatelogs：日志滚动工具；
5. suexec：访问某些有特殊权限配置的资源时，临时切换至指定用户身份运行；
6. ab： apache benchmark

### 4.1 httpd的压力测试工具
- ab, webbench, http_load, seige
- jmeter, loadrunner
- tcpcopy：网易，复制生产环境中的真实请求，并将之保存下来；

ab  [OPTIONS]  URL
- -n：总请求数；
- -c：模拟的并行数；
- -k：以持久连接模式 测试；
- - 附注: ulimit -n num - 调整当前用户能同时打开的文件数
