# 22.2 mysql的安装配置
上一节我们对关系型数据库和 mariadb 做了一个简单介绍，接下来我们来学习 mariadb 的安装配置

##  1. Mariadb 配置
### 1.1 配置文件格式

mysql 的配置文件是 ini 风格的配置文件；客户端和服务器端的多个程序可通过一个配置文件进行配置，使用 `[program_name]` 标识配置的程序即可。

```
vim /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

# include all files from the config directory
!includedir /etc/my.cnf.d
```

### 1.2 配置文件读取次序
mysql 的各类程序启动时都读取不止一个配置文件，配置文件将按照特定的顺序读取，最后读取的为最终生效的配置。可以使用 `my_print_defaults` 查看默认的配置文件查找次序。

```
$ my_print_defaults
Default options are read from the following files in the given order:
/etc/mysql/my.cnf /etc/my.cnf ~/.my.cnf
```

#### 配置文件查找次序        
默认情况下 OS Vendor提供mariadb rpm包安装的服务的配置文件查找次序：  
1. `/etc/mysql/my.cnf`
2. `/etc/my.cnf`
3. `/etc/my.cnf.d/`
4. `--default-extra-file=/PATH/TO/CONF_FILE`: 通过命令行指定的配置文件
5. `~/.my.cnf`: 家目录下的配置文件

通用二进制格式安装的服务程序其配置文件查找次序
2. `/etc/my.cnf`
3. `/etc/my.cnf.d/`
1. `/etc/mysql/my.cnf`
4. `--default-extra-file=/PATH/TO/CONF_FILE`: 通过命令行指定的配置文件
5. `~/.my.cnf`: 家目录下的配置文件

```
# os rpm 包安装的 mariadb 配置文件
ll -d /etc/my*
-rw-r--r--. 1 root root 570 6月   8 2017 /etc/my.cnf
drwxr-xr-x. 2 root root  67 2月  27 09:57 /etc/my.cnf.d

ll /etc/my.cnf.d
总用量 12
-rw-r--r--. 1 root root 295 4月  30 2017 client.cnf
-rw-r--r--. 1 root root 232 4月  30 2017 mysql-clients.cnf
-rw-r--r--. 1 root root 744 4月  30 2017 server.cnf
```


### 1.3 初始化配置
mysql的用户账号由两部分组成：`'USERNAME'@'HOST'`; `HOST`: 用于限制此用户可通过哪些远程主机连接当前的mysql服务.HOST的表示方式，支持使用通配符：
- `%`：匹配任意长度的任意字符；
- `172.16.%.%` == `172.16.0.0/16`
- `_`：匹配任意单个字符；

默认情况下 mysql 登陆时会对客户端的 IP 地址进行反解，这种反解一是浪费时间可能导致阻塞，二是如果反解成功而 mysql 在授权时只授权了 IP 地址而没有授权主机名，依旧无法登陆，所以在配置 mysql 时都要关闭名称反解功能。

```
vim  /etc/mysql/my.cnf  # 添加三个选项：
datadir = /mydata/data
innodb_file_per_table = ON
skip_name_resolve = ON
```

### 1.4 mysql 安全初始化
默认安装的情况下 mysql root 帐户是没有密码的，可通过 mysql 提供的安全初始化脚本，快速进行安全初始化。
```
# 查看mysql用户及其密码
mysql
> use mysql;
> select user,host,password from user;

# 运行脚本安全初始化脚本
/user/local/mysql/bin/mysql_secure_installation
```


## 2. MariaDB 安装
常见的安装方式有如下三种:
1. rpm包；由OS的发行商提供，或从程序官方直接下载
2. 源码包编译安装: 编译安装，除非需要定制功能，否则一般不推荐编译安装
3. 通用二进制格式的程序包: 展开至特定路径，并经过简单配置后即可使用，这种方式便于部署，无需解决环境依赖

### 2.1 二进制程序包安装
Centos 6：
- 准备数据目录；以/mydata/data目录为例；
- 安装配置mariadb

```                
groupadd -r -g 306 mysql
useradd  -r  -g 306 -u 306 mysql
tar xf  mariadb-VERSION.tar.xz  -C  /usr/local
cd /usr/local
ln  -sv  mariadb-VERSION  mysql
cd  /usr/local/mysql
chown  -R  root:mysql  ./*
scripts/mysql_install_db  --user=mysql  -datadir=/mydata/data
cp  support-files/mysql.server  /etc/init.d/mysqld
chkconfig  --add  mysqld
chkconfig  --list mysqld

# 跳过名称解析，并进行安全初始化
```
