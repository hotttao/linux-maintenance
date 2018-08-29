# 25.2 日志管理系统rsyslog
rsyslog 是Linux 系统上日志管理系统，应用程序可直接调用 rsyslog 的接口将日志写入到 rsyslog 特定的 facility 中即可完成日志记录。如果应用程序通过 rsyslog 来记录日志，通常在其自己的配置文件中有专门的选项用来定义将日志存入到 rsyslog 哪个 facility。facility 可以理解为 rsyslog 日志收集器的基本单元，rsyslog 内部的配置文件定义了每个 facility 的日志存储于何处。应用程序只需要将日志信息教给 rsyslog，rsyslog 会自动根据日志所属的 facility 将日志存储到对应的位置。本节我们就来详细介绍 rsyslog 的配置使用

## 1. rsyslog 简介
### 1.1 syslog
syslog 是 rsyslog 的上一版，syslog 服务分成了两个部分:
1. syslogd： system，为 Linux 上的应用程序提供日志记录服务
2. klogd：kernel，为开机启动，系统内核提供日志记录服务

除了本地服务，syslog 还支持C/S架构，即可通过UDP或TCP协议为网络上的其他主机提供日志记录服务。这种模式下

```
# C/S rsyslog
                    --------
kernel  -------->   |本机    |
                    |       |     tcp/utp
ssh     -------->   |rsyslog|  ------------>   rsyslog  server
                    |       |
...     --------->  |服务    |
                    ---------
```

1. 作为 server 的 syslog 服务监听在 tcp/utp 的 514 端口上
2. 作为客户端的应用程序，首先将日志发送到本地的 syslog 服务上，再由 本地的 syslog 服务作为客户端将应用程序的日志发送到 server 端的 syslog 上加以记录，因此 syslog 的客户端与服务器都是 syslog
3. rsyslog  server 收到客户端发来的日志后，根据自己 facility 的配置将日志记录到特定位置。

因此应用程序只是将日志写入到特定的 facility，syslog server 只是本地 syslog 记录日志的一种方式。syslog 可定义 facility 的日志存储方式，可以是本地文件，也可以是远程的 syslog server

syslog 日志格式无法自定义，统一为`事件产生的日期时间 	主机 	进程[pid] ：事件内容`，因此只能记录一些简单的日志。


### 1.2 rsyslog
rsyslog 是 syslog 的升级版本，支持所有 syslog 特定，它只有 rsyslogd 一个服务来完成所有日志的记录功能。相比于 syslog，rsyslog 具有如下新特性:
1. 支持多线程；
2. 支持多种C/S连接协议，UDP，TCP，SSL，TLS，RELP；
3. 可存储日志信息于MySQL、PGSQL、Oracle等数据管理系统；
4. 强大的过滤器，实现过滤日志信息中任何部分的内容；
5. 自定义输出格式

## 2. rsyslog 组成
### 2.1 日志收集器单元
rsyslog日志收集器有两个重要的概念:
1. `facility`：
  - 作用: 设施，从功能或程序上对日志收集进行分类
  - 内置: rsyslog 上默认的 facility 有 `auth, authpriv, cron, daemon, kern, lpr, mail, mark, news, security, user, uucp, local0-local7, syslog`
2. `priority`：
  - 作用: 日志级别，用于定义日志的重要性，facility 可定义记录日志的级别范围
  - 级别: 日志级别从低到高有 `debug, info, notice, warn(warning), err(error), crit(critical), alert, emerg(panic)`


### 2.2 程序组成
```
$ rpm -ql rsyslog
/etc/logrotate.d/syslog
/etc/pki/rsyslog
/etc/rsyslog.conf                       # 配置文件
/etc/rsyslog.d

/etc/sysconfig/rsyslog                  # rsyslog 服务的配置文件
/usr/bin/rsyslog-recover-qi.pl

/usr/lib/systemd/system/rsyslog.service # 服务脚本
/usr/lib64/rsyslog                      # 模块目录
/usr/lib64/rsyslog/im*.so               # im 开头的为输入相关模块
/usr/lib64/rsyslog/om*.so               # om 开头的为输出相关模块
/usr/lib64/rsyslog/lm*.so
/usr/lib64/rsyslog/mm*.so
/usr/lib64/rsyslog/pm*.so
```

## 3. rsyslog 配置
### 3.1 配置文件结构
```
$ cat /etc/rsyslog.conf |grep -v "^# "

#### MODULES ####               # 模块加载
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imjournal # provides access to the systemd journal
#$ModLoad imklog # reads kernel messages (the same are read from journald)
#$ModLoad immark  # provides --MARK-- message capability

$ModLoad imudp                  # utp 服务
$UDPServerRun 514

$ModLoad imtcp                 # tcp 服务
$InputTCPServerRun 514

#### GLOBAL DIRECTIVES ####     # 全局目录配置
$WorkDirectory /var/lib/rsyslog
$IncludeConfig /etc/rsyslog.d/*.conf

#### RULES ####                 # facility 日志记录配置
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
mail.*                                                  -/var/log/maillog
*.emerg                                                 :omusrmsg:*
```
配置文件由三个部分组成
1. `MODULES`: 模块加载
2. `GLOBAL DIRECTIVES`: 全局变量
3. `RULES`: 用于定义 facility 记录日志的级别和位置，格式为 `facilty.priority 	target`

### 1.2 RULES 格式
RULES 用于定义 facility 记录日志的级别和位置，其语法为

`facilty.priority 	target`
- `priority`: 日志级别，有如下几种表示方式
  - `*`：所有级别；
	- `none`：没有级别；
	- `priority`：此级别以高于此级别的所有级别；
	- `=priorty`：仅此级别
- `target`: 日志输出的位置，有如下几种格式
  - `/var/log/messages`: 记录到特定文件中，默认为同步写入，大量日志记录会拖慢系统性能
  - `-/var/log/maillog`: 记录到文件，`-` 表示异步写入，不重要的日志可异步写入，减少系统 IO
  - `:omusrmsg:tao`: 调用 `omusrmsg` 将日志发送到用户登陆的终端，`*` 表示所有登陆用户；
  - `@192.168.1.101`: 将日志发送到 rsyslog server
  - `| COMMAND`: 将日志送入管道
- 说明: `target` 中可使用 `:module:param` 调用 rsyslog 内置的模块，每个模块有自己特定的参数

### 1.3 默认 facilty
```
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
authpriv.*                                              /var/log/secure
mail.*                                                  -/var/log/maillog
cron.*                                                  /var/log/cron
*.emerg                                                 :omusrmsg:*
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
local2.*                                                /var/log/haproxy.log
```

### 1.4 其他日志文件
除了 rsyslog 记录的日志外，系统上还有其他一些重要的日志文件
- `/var/log/wtmp`：
  - 作用: 当前系统成功登录系统的日志
  - 查看: `last` 命令
- `/var/log/btmp`：
  - 作用: 当前系统尝试登录系统失败相关的日志
  - 查看: `lastb`命令
  - 附注: `lastlog`命令，能显示当前系统上的所有用户最近一次登录系统的时间；
- `/var/log/dmesg`：
  - 作用: 系统引导过程中的日志信息
  - 查看: 也可以使用dmesg命令进行查看


## 4. rsyslog 高级配置
### 4.1 rsyslog server 配置
配置 C/S 架构的 rsyslog 步骤如下所示

```
# 1. 服务器端: 启动 rsyslog server 监听 tcp/udp 的模块
# Provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514

# 2. 客户端: 配置 facility 将日志发往服务端
*.info;mail.none;authpriv.none;cron.none     @192.168.1.149
```

### 4.2 记录日志于mysql中
记录日志于mysql中首先要安装配置 rsyslog mysql 的模块，配置步骤如下:
```
# 1. rsyslog mysql 模块安装
$ yum  search  rsyslog
$ sudo yum install rsyslog-mysql.x86_64

# 2. mysql 配置
$ rpm -ql rsyslog-mysql.x86_64
/usr/lib64/rsyslog/ommysql.so
/usr/share/doc/rsyslog-8.24.0/mysql-createDB.sql

# 通过导入createDB.sql脚本创建依赖到的数据库及表
$ mysql -uUSER  -hHOST  -pPASSWORD  < /usr/share/doc/rsyslog-mysql-VERSION/createDB.sql

# 登陆 mysql 配置 rsyslog 使用的特定帐户
$ mysql -uUSER  -hHOST  -pPASSWORD
MariaDB [(none)]> grant all on Syslog.* to "rsyslog"@"%" identified by "rsyspass";
MariaDB [(none)]> flush privileges;

mysql -ursyslog -prsyspass
>

# 3. 配置rsyslog使用ommysql模块
$ sudo vim /etc/rsyslog.conf
### MODULES ####
$ModLoad  ommysql

#### RULES ####
facility.priority 		:ommysql:DBHOST,DB,DBUSER,DBUSERPASS

# 重启rsyslog服务
$ sudo systemctl restart rsyslog
```
