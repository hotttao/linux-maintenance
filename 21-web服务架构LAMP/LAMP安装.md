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
