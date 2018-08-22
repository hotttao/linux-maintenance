# 21.3 LAMP安装
前面我们我们介绍了 LAMP 的原理部分，本节我们就来实践，搭建一个 LAMP。


## 1. php 配置
### 1.1 httpd 与 php 结合方式
前面我们介绍 LAMP 的基本原理时提到过，httpd 与 php 有三种结合方式
1. CGI: 由 httpd 服务创建子进程来加载和执行 php 脚本
2. fpm（FastCGI Process Manager): php 进程管里器，将 php 的解析执行作为独立的应用程序服务器
3. modules: 将 php编译成为 httpd 的模块，httpd 既是 web 服务器也是应用程序服务器
    - `prefork` MPM 下需要加载 `libphp5.so` 模块
    - `event, worker` MPM 下需要加载 `libphp5-zts.so` 模块

#### modules
将 php 作为 http 的 modules 由 `php` 包提供  
```
$ yum info php

$ rpm -ql php
/etc/httpd/conf.d/php.conf
/etc/httpd/conf.modules.d/10-php.conf
/usr/lib64/httpd/modules/libphp5.so    # prefork MPM 的 php 所需模块
/usr/share/httpd/icons/php.gif
/var/lib/php/session
```

#### fpm
fpm 由 `php-fpm` 包提供
```
$ yum info php-fpm
$ rpm -ql php-fpm
/etc/logrotate.d/php-fpm
/etc/php-fpm.conf       # php-fpm 服务的配置文件
/etc/php-fpm.d
/etc/php-fpm.d/www.conf
/etc/sysconfig/php-fpm
/run/php-fpm
/usr/lib/systemd/system/php-fpm.service
/usr/lib/tmpfiles.d/php-fpm.conf
/usr/sbin/php-fpm
```

### 1.2 php 相关包
与 php 相关的 rpm 包有如下几个:
1. `php`: 实现 php 作为 httpd 的一个模块
2. `php-fpm`: fpm
3. `php-common`: php 的核心文件
4. `php-mysql`: php 的 mysql 驱动模块
5. `php-xcache`: php 的加速器

## 1.3 php核心文件
```
$ rpm -ql php-common
/etc/php.ini            # 配置文件
/etc/php.d
/etc/php.d/curl.ini
/etc/php.d/fileinfo.ini
/etc/php.d/json.ini
/etc/php.d/phar.ini
/etc/php.d/zip.ini
/usr/lib64/php
/usr/share/php
/var/lib/php
```
php 的所有核心文件均由 `php-common` 包提供，配置文件为:
- `/etc/php.ini`
- `/etc/php.d/*.ini`

php 的配置文件在 php 启动时被读取一次
- 对于服务器模块存在的 php 仅在web 服务器启动时读取一次
    - Modules：重启httpd服务生效；
    - FastCGI：重启php-fpm服务生效；
- 对于cgi 和 cli 版本，每次调用都会读取

#### php.ini
php 的文档参考如下:
- php.ini的核心配置选项文档：  http://php.net/manual/zh/ini.core.php
- php.ini配置选项列表：http://php.net/manual/zh/ini.list.php
- 注释符：
    - 较新的版本中，已经完全使用`;`进行注释；
    - `#`：纯粹的注释信息
    - `;`：用于注释可启用的directive

```
# 配置方式类似 yum.respo.d, 采用分段进行
# ;(分号) 表示注释符
[foo]：Section Header
directive = value
```

### 1.3 php xcache 加速器
```
# 安装
yum install php-xcache

# 配置
/etc/php.d/xcache.ini
```

## 2. modules 模式的 LAMP 安装
首先我们来介绍将 php 作为 httpd 的一作模块这种模式下 LAMP 的安装配置。安装完成后 php 的配置文件位于 `/etc/httpd/conf.d/php.conf`

### 2.1 Centos 6
Centos 下需要安装 `httpd, php, php-mysql, mysql-server`，然后启动 httpd 和 mysql 服务

```
yum install -y httpd php php-mysql mysql-server
service httpd start
service mysqld star
```

### 2.2 Centos 7
Centos7 下需要安装 `httpd, php, php-mysql, mariadb-server`。需要注意的是 php 在不同的 MPM 下安装的方式不一样，默认 `yum install php` 安装要求 httpd 使用 prefork MPM。

```
# 1. 安装 LAMP
yum install http  php php-mysql  mariadb-server
systemctl start httpd
systemctl start mariadb

# 2. php 的配置
$ rpm -ql php
/etc/httpd/conf.d/php.conf
/etc/httpd/conf.modules.d/10-php.conf
/usr/lib64/httpd/modules/libphp5.so    # prefork MPM 的 php 所需模块
/usr/share/httpd/icons/php.gif
/var/lib/php/session

```

### 2.3 测试
#### php 程序执行环境
```
<?php
    phpinfo()
?>
```

#### php 与mysql 通信
```
# vim DocumentRoot/a.php
<?php
    $con=mysql_connect('127.0.0.1','','');
    if ($con)
        echo "OK";
    else
        echo "faile";
    mysql_close();
    phpinfo();
?>
```

## 3. fpm 的 LAMP 安装
### 3.1 Centos6
- PHP-5.3.2：默认不支持fpm机制；需要自行打补丁并编译安装；
- httpd-2.2：默认不支持fcgi协议，需要自行编译此模块；
- 解决方案：编译安装httpd-2.4, php-5.3.3+；

### 3.2 Centos7
- httpd-2.4：rpm包默认编译支持了fcgi模块；
- php-fpm包：专用于将php运行于fpm模式；

#### 安装 msyql
```
# 1. 安装 msyql
yum isntall -y  mariadb-server
vim /etc/my.cnf.d/server.cnf
  [mysqld]
  skip_name_resolve=ON
  innodb_file_per_table=ON

# 安全初始化
mysql_secure_installation

# 创建普通登陆用户
# mysql -uroot -p
mysql> grant all on testdb.* to 'myuser'@'172.16.0.%' indentified by "mypass"

mysql> flush priviledges
```

#### 安装 php-fpm
```
# 2. 安装 php-fpm，最好不要与 php 包同时安装
yum install php-fpm php-mysql php-mbstring

$ rpm -ql php-fpm
/etc/logrotate.d/php-fpm
/etc/php-fpm.conf
/etc/php-fpm.d
/etc/php-fpm.d/www.conf

# 配置 php-fpm
vim /etc/php-fpm.conf
vim /etc/php-fpm.d/www.conf
  pm = static|dynamic
    # - static：固定数量的子进程；
    #   - pm.max_children；
    # - dynamic：子进程数据以动态模式管理；
    #   - pm.start_servers
    #   - pm.min_spare_servers
    #   - pm.max_spare_servers
    #   - ;pm.max_requests = 500

# 创建session目录，并确保运行php-fpm进程的用户对此目录有读写权限；
mkdir  /var/lib/php/session
chown apache.apache /var/lib/php/session
```

#### 配置 httpd
```
# 1. 确定是否启用httpd的相关模块
httpd -M|grep proxy_module

# 未启用则启用代理模块
vim /etc/httpd/httpd.conf  
    > LoadModule proxy_module modules/mod_proxy.so
    > LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so

# 2. 配置虚拟主机支持使用fcgi
# 在相应的虚拟主机中添加类似如下两行。
    > ProxyRequests Off
    > ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/PATH/TO/DOCUMENT_ROOT/$1

# ProxyRequests Off：关闭正向代理
# ProxyPassMatch：把以.php结尾的文件请求发送到php-fpm进程，php-fpm至少需要知道运行的目录和URI，所以这里直接在fcgi://127.0.0.1:9000后指明了这两个参数，其它的参数的传递已经被mod_proxy_fcgi.so进行了封装，不需要手动指定。

# 虚拟主机配置示例
DirectoryIndex index.php

<VirtualHost *:80>
    ServerName www.b.net
    DocumentRoot /apps/vhosts/b.net
    ProxyRequests Off
    ProxyPassMatch ^/(.*\.php)$  fcgi://127.0.0.1:9000/apps/vhosts/b.net/$1

    <Directory "/apps/vhosts/b.net">
        Options None
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>  
```

#### 安装 php xcache
```
yum install -y php-xcache

$ rpm -ql php-xcache
/etc/php.d/xcache.ini
/usr/lib64/php/modules/xcache.so
```
