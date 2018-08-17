# 21.4 httpd2.4 基础配置
上一节我们详细介绍了httpd2.2 的配置，对比着 httpd2.2 本节我们来讲解 httpd2.4 的配置。

## 1. httpd-2.4
### 1.1 新特性
相比于 httpd2.2 httpd2.4 有如下新特性:
- **MPM支持运行为DSO机制**；以模块形式按需加载；
- event MPM生产环境可用；
- 异步读写机制；
- 支持每模块及每目录的单独日志级别定义；
- 每请求相关的专用配置: <if>
- 增强版的表达式分析式；
- 毫秒级持久连接时长定义；
- 基于FQDN的虚拟主机也不再需要 **NameVirutalHost** 指令；
- 新指令，AllowOverrideList；
- 支持用户自定义变量；
- 新模块：
  - mod_proxy_fcgi
  - mod_raltelimit
  - mod_remoteip


### 1.3 配置文件：
1. 主配置文件: `/etc/httpd/conf/httpd.conf`
2. 模块配置文件: `/etc/httpd/conf.modules.d/*.conf`
    - mpm 以DSO机制提供，配置文件为 `00-mpm.conf`
3. 辅助配置文件: `/etc/httpd/conf.d/*.conf`

### 1.4 配置应用：
1. 切换使用的MPM
    - 编辑配置文件`/etc/httpd/conf.modules.d/00-mpm.conf`，启用要启用的MPM相关的LoadModule指令即可。
2. 基于IP的访问控制
    - 变更:
        - 新增访问路径必须添加 Require 进行 ip 授权，否则新增路径不允许访问
        - 所有的IP 访问控制必须放置在 RequireAll 容器中
    - 允许所有主机访问：`Require  all  granted`
    - 拒绝所有主机访问：`Require  all  deny`
    - 控制特定的IP访问：
        - `Require  ip  IPADDR`：授权指定来源的IP访问；
        - `Require  not  ip  IPADDR`：拒绝
        - IPADDR：
            - IP
            - NetAddr: 子网
                - 172.16
                - 172.16.0.0
                - 172.16.0.0/16
                - 172.16.0.0/255.255.0.0
    - 控制特定的主机访问：
        - `Require  host  HOSTNAME`：授权指定来源的主机访问；
        - `Require  not  host  HOSTNAME`：拒绝
        - HOSTNAME：
            - FQDN：特定主机
            - domin.tld：指定域名下的所有主机              
3. 虚拟主机
    - 基于FQDN的虚拟主机也不再需要NameVirutalHost指令；
    - 注意：任意目录下的页面只有显式授权才能被访问；
4. ssl
5. KeepAliveTimeout  #ms: 毫秒级持久连接时长定义；

```
# IP 访问控制                                          
<RequireAll>
    Require all granted
    Require not ip 172.16.100.2
</RequireAll>  

# 定义虚拟主机
<VirtualHost *:80>
    ServerName www.b.net
    DocumentRoot "/apps/b.net/htdocs"
    <Directory "/apps/b.net/htdocs">
        Options None
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>                        
```

### 1.5 练习题：分别使用httpd-2.2和httpd-2.4实现；
```
1. 建立httpd服务，要求：
    (1) 提供两个基于名称的虚拟主机：
        www1.stuX.com，页面文件目录为/web/vhosts/www1；错误日志为/var/log/httpd/www1/error_log，访问日志为/var/log/httpd/www1/access_log；
        www2.stuX.com，页面文件目录为/web/vhosts/www2；错误日志为/var/log/httpd/www2/error_log，访问日志为/var/log/httpd/www2/access_log；
    (2) 通过www1.stuX.com/server-status输出其状态信息，且要求只允许提供账号的用户访问；
    (3) www1不允许192.168.1.0/24网络中的主机访问；        
2. 为上面的第2个虚拟主机提供https服务，使得用户可以通过https安全的访问此web站点；
    (1) 要求使用证书认证，证书中要求使用国家（CN），州（Beijing），城市（Beijing），组织为(MageEdu)；
    (2) 设置部门为Ops, 主机名为www2.stuX.com；
```
