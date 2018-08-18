# 21.7 httpd 编译安装

### 1.2 安装httpd-2.4
依赖:
- apr-1.4+, apr-util-1.4+, [apr-iconv]
- apr： apache portable runtime

#### CentOS 6：
- 默认：apr-1.3.9, apr-util-1.3.9, http-2.4 需要编译安装
- 准备:
    - 开发环境包组：Development Tools, Server Platform Development
    - 开发程序包：pcre-devel
- 自带的服务控制脚本：apachectl
- 编译安装步骤：
```
# apr-1.4+
./configure  --prefix=/usr/local/apr
make && make install

# apr-util-1.4+
./configure  --prefix=/usr/local/apr-util  --with-apr=/usr/local/apr
make && make install

# httpd-2.4
groupadd -r apache
useradd -r -g apache apache
yum install pcre-devel -y
./configure --prefix=/usr/local/apache24 --sysconf=/etc/httpd24  --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --enable-modules=most --enable-mpms-shared=all --with-mpm=prefork
make  && make install

# 服务管理
/usr/local/apache/bin/apachectl {start|stop|restart}

# 关闭防火墙和SELinux
iptables -F
setenforce 0

# 编写 initd 开机启动脚本
cp /etc/init.d/http /etc/init.d/http24
vim /etc/init.d/http24 # 更改相应变量
chkconfig --add http24
chkconfig --list http24
chkconfig http24 on
```                          
