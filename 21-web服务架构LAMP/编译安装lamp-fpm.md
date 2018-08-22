# 21.7 编译安装 lamp-fpm

## 1. Centos6 编译安装 lamp-fpm
### 1.1 编译安装步骤
1. httpd：编译安装，httpd-2.4
3. mairadb：通用二进制格式，mariadb-5.5
2. php5：编译安装，php-5.4
5. xchache
4. 注意：任何一个程序包被编译操作依赖到时，需要安装此程序包的“开发”组件，其包名一般类似于name-devel-VERSION；

### 1.2 编译安装apache
```
# 准备开发环境
yum groupinstall "Development Tools" "Server Platform Development" -y

# 1. 编译安装apr
tar xf apr-1.5.0.tar.bz2
cd apr-1.5.0
./configure --prefix=/usr/local/apr
make && make install


# 2. 编译安装apr-util
tar xf apr-util-1.5.3.tar.bz2
cd apr-util-1.5.3
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
make && make install


# 3. httpd-2.4.9编译过程也要依赖于pcre-devel软件包，需要事先安装
yum install pcre-devel -y

# 4. 编译安装httpd-2.4.9
tar xf httpd-2.4.9.tar.bz2
cd httpd-2.4.9
./configure --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --enable-modules=most --enable-mpms-shared=all --with-mpm=event --prefix=/usr/local/apache --sysconfdir=/etc/httpd24 --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util
make && make install

# 5. 提供SysV服务脚本/etc/rc.d/init.d/httpd
cd /etc/rc.d/init.d
cp httpd httpd24
vim http24
    > apachectl=/usr/local/apache/bin/apachectl
    > httpd=${HTTPD-/usr/local/apache/bin/httpd}
    > pidfile=
    > logfile=
chkconfig --add httpd24
chkconfig --list httpd24

# 6. 配置系统环境
vim /etc/profile.d/httpd24.sh
    > export PATH=/usr/local/apache/bin:$PATH
. /etc/profile.d/httpd24.sh
httpd -t
service start httpd24

# 7. 修改配置文件
cd  /etc/httpd24
vim httpd.conf
    > PidFile  "/var/run/httpd.pid"  
# 说明:
# httpd.conf 中的 PidFile 必须与 init 服务启动脚本中的pidfile 保持一致
# 默认 httpd.conf 的PidFile=/usr/local/apache/logs/httpd.pid
```

### 1.3 编译安装 Mariadb
```
# 1. 准备数据存放的文件系统
# 新建一个逻辑卷，并将其挂载至特定目录即可
# 假设挂载目录为/mydata，/mydata/data 为mysql数据的存放目录

# 2. 新建用户以安全方式运行进程：
groupadd -r mysql
useradd -g mysql -r -s /sbin/nologin -M -d /mydata/data mysql
chown -R mysql:mysql /mydata/data

# 3. 安装并初始化mariadb
# 下载通用二进制包 mariadb-5.5.60-linux-systemd-x86_64.tar.gz
tar xf mysql-5.5.33-linux2.6-i686.tar.gz -C /usr/local
cd /usr/local/
ln -sv mysql-5.5.33-linux2.6-i686  mysql
cd mysql

chown -R mysql:mysql  .
scripts/mysql_install_db --user=mysql --datadir=/mydata/data
chown -R root  .

# 4. 为mysql提供主配置文件：
cd /usr/local/mysql
cp support-files/my-large.cnf  /etc/my.cnf
vim /etc/my.cnf
    > thread_concurrency = cpu * 2
    > datadir = /mydata/data
    > innodb_file_per_table = on
    > skip_name_resolve = on

# 5. 为mysql提供sysv服务脚本：
cd /usr/local/mysql
cp support-files/mysql.server  /etc/rc.d/init.d/mysqld
chmod +x /etc/rc.d/init.d/mysqld

chkconfig --add mysqld
chkconfig mysqld on
service start mysqld

# 6. 删除 mysql 匿名用户
cd /usr/local/mysql
scripts/mysql_secure_installation

# 7. man，PATH 环境变量
# 输出mysql的man手册至man命令的查找路径
vim /etc/man.config
    > MANPATH  /usr/local/mysql/man

# 输出mysql的头文件至系统头文件路径/usr/include
ln -sv /usr/local/mysql/include /usr/include/mysql

# 输出mysql的库文件给系统库查找路径
echo '/usr/local/mysql/lib' > /etc/ld.so.conf.d/mysql.conf
ldconfig  # 让系统重新载入库

# 修改PATH环境变量
vim /etc/profile.d/mysql.sh
    > export PATH=/usr/local/mysql/bin:$PATH
```

### 1.4 编译安装 php
```
# 1. 解决依赖关系：
yum -y groupinstall "X Software Development"

# 如果想让编译的php支持mcrypt扩展
yum install libmcrypt
yum install libmcrypt-devel
yum install mhash
yum install mhash-devel

# 2. 编译安装 php
tar xf php-5.4.26.tar.bz2
cd php-5.4.26
./configure --prefix=/usr/local/php5 --with-mysql=/usr/local/mysql --with-openssl --with-mysqli=/usr/local/mysql/bin/mysql_config --enable-mbstring --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml  --enable-sockets --enable-fpm --with-mcrypt  --with-config-file-path=/etc --with-config-file-scan-dir=/etc/php.d --with-bz2
# 说明：
    # fpm 必须启用 --enable-fpm
    # 需要更改的配置有
    # --prefix=/usr/local/php5 --with-mysql=/usr/local/mysql --with-mysqli=/usr/local/mysql/bin/mysql_config --with-config-file-path=/etc --with-config-file-scan-dir=/etc/php.d

make
make intall

# 3. 为php提供配置文件：
cp php.ini-production /etc/php.ini

# 4. 为php-fpm提供SysV init脚本，并将其添加至服务列表：
cp sapi/fpm/init.d.php-fpm  /etc/rc.d/init.d/php-fpm
chmod +x /etc/rc.d/init.d/php-fpm
chkconfig --add php-fpm
chkconfig php-fpm on

# 5. 为php-fpm提供配置文件：
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf

# 配置fpm的相关选项为你所需要的值，并启用pid文件（如下最后一行）：
vim /usr/local/php/etc/php-fpm.conf
    > pm.max_children = 50
    > pm.start_servers = 5
    > pm.min_spare_servers = 2
    > pm.max_spare_servers = 8
    > pid = /usr/local/php/var/run/php-fpm.pid # 与 init 脚本一致

# 启动php-fpm, 默认情况下，fpm监听在127.0.0.1的9000端口
service php-fpm start  
ps aux | grep php-fpm
```    

### 1.5 配置 httpd
```
# 1. 启用httpd的相关模块
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


# 3. 让apache能识别php格式的页面，并支持php格式的主页,并支持php格式的主页
vim /etc/httpd/httpd.conf
    > AddType application/x-httpd-php  .php
    > AddType application/x-httpd-php-source  .phps
    > DirectoryIndex  index.php  index.html
```

### 1.6 安装 xcache
```
# 1. 安装
tar xf xcache-3.0.3.tar.gz
cd xcache-3.0.3
/usr/local/php/bin/phpize
./configure --enable-xcache --with-php-config=/usr/local/php/bin/php-config
make && make install

# 安装结束时，会出现类似如下行：
# Installing shared extensions: /usr/local/php/lib/php/extensions/no-debug-zts-20100525/

cp xcache.ini /etc/php.d
vim /etc/php.d/xcache.ini
    > extension="上述安装结束提示的路径"
```

### 1.7 php-fpm 配置
`vim /usr/local/php/etc/php-fpm.conf`

`pm = static|dynamic`
- static：固定数量的子进程；
    - pm.max_children；
- dynamic：子进程数据以动态模式管理；
    - pm.start_servers
    - pm.min_spare_servers
    - pm.max_spare_servers
    - ;pm.max_requests = 500

创建session目录，并确保运行php-fpm进程的用户对此目录有读写权限；
```
# mkdir  /var/lib/php/session
# chown apache.apache /var/lib/php/session
```
