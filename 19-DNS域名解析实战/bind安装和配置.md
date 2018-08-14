# 19.3 bind安装和配置
bind 全称为 Berkeley Internet Name Domain，是 DNS 协议的一种开源实现。由伯克利分校开发，现由 ISC 维护，也是现在使用最为广泛的DNS服务器软件，本节我们就来介绍如何使用 bind 配置一个 DNS 服务器。

## 1. BIND
### 1.1 bind 安装
#### rpm 包组成
```
$ yum install bind
$ yum list all bind*
已安装的软件包
bind.x86_64                   32:9.9.4-61.el7      @base
bind-libs.x86_64              32:9.9.4-61.el7      @base
bind-libs-lite.x86_64         32:9.9.4-61.el7      @base
bind-license.noarch           32:9.9.4-61.el7      @base
bind-utils.x86_64             32:9.9.4-61.el7      @base
```
bind 的 rpm 包主要由以下几个:
- `bind`：提供的dns server程序、以及几个常用的测试程序；
- `bind-utils`：bind客户端程序集，例如dig, host, nslookup等，可用于测试 dns 服务；
- `bind-libs`：被bind和bind-utils包中的程序共同用到的库文件；
- `bind-chroot`：选装，让named运行于jail模式下,目的是限定 bind 的运行环境，更加安全

#### 程序文件
```bash
$ rpm -ql bind|grep sbin
/usr/sbin/arpaname
/usr/sbin/ddns-confgen
/usr/sbin/dnssec-checkds      # DNS 安全扩展
/usr/sbin/dnssec-coverage
/usr/sbin/dnssec-dsfromkey
/usr/sbin/dnssec-importkey
/usr/sbin/dnssec-keyfromlabel
/usr/sbin/dnssec-keygen
/usr/sbin/dnssec-revoke
/usr/sbin/dnssec-settime
/usr/sbin/dnssec-signzone
/usr/sbin/dnssec-verify
/usr/sbin/genrandom
/usr/sbin/isc-hmac-fixup
/usr/sbin/lwresd
/usr/sbin/named                # bind serve 进程
/usr/sbin/named-checkconf      # bind 配置文件检查
/usr/sbin/named-checkzone      # DNS 数据库区域文件检查
/usr/sbin/named-compilezone
/usr/sbin/named-journalprint
/usr/sbin/nsec3hash
/usr/sbin/rndc                # bind 的远程控制工具
/usr/sbin/rndc-confgen
```
bind rpm 包提供了以下核心程序:
- `dnssec-*`: Domain Name System Security Extensions,DNS 安全扩展
- `rndc`: named 进程远程控制工具
- `named`: bind server 的核心程序
- `named-checkconf`: 用于检查 named 配置文件是否存在语法错误
- `named-checkzone`: 用于检查 DNS 的区域解析库文件是否存在语法错误

#### named-checkconf
`named-checkconf [named.conf]`
- 作用: 检查 named 配置文件是否存在语法错误
- 参数: named.conf 配置文件位置，默认为 `/etc/named.conf`

#### named-checkzone
`named-checkzone zonename filename`
- 作用: 用于检查 DNS 的区域解析库文件是否存在语法错误
- 参数:
	- `zonename`: 区域名称
	- `filename`: 区域数据库文件所在位置

```
$ named-checkzone magedu.com.  /var/named/magedu.com.zone
```

## 1.2 bind 配置文件
```bash
$ rpm -ql bind|egrep "etc|var"|grep -v "share"
/etc/logrotate.d/named
/etc/named                 
/etc/named.conf            # 核心配置文件
/etc/named.iscdlv.key      # 核心配置文件内使用 include 包含的辅助配置文件
/etc/named.rfc1912.zones
/etc/named.root.key
/etc/rndc.conf             # rndc 配置文件
/etc/rndc.key
/etc/rwtab.d/named
/etc/sysconfig/named
/var/log/named.log        # 日志文件
/var/named                # DNS 数据库区域解析文件默认所在的目录
/var/named/data
/var/named/dynamic
/var/named/named.ca       # 顶级域的配置文件
/var/named/named.empty
/var/named/named.localhost
/var/named/named.loopback
/var/named/slaves
```
bind 的配置文件包括两个部分:
1. 主配置文件：`/etc/named.conf`
2. 主配置文件内使用 include 包含进来辅助配置文件:
	- `/etc/named.iscdlv.key`
	- `/etc/named.rfc1912.zones`
	- `/etc/named.root.key`

解析库文件默认存放在 `/var/named/`目录下；一般名字为：ZONE_NAME.zone。一台DNS服务器可同时为多个区域提供解析，且必需包含如下几个区域解析库文件:
1. 根区域解析库文件： `named.ca`
2. localhost 的正向解析库：`named.localhost`
3. 127.0.0.1 的反向解析库：`named.loopback`

默认情况下，上述的区域解析库文件在 bind 安装时，已由 rpm 包自动提供。

### 1.3 bind 进程管理
```
$ rpm -ql bind|grep systemd
/usr/lib/systemd/system/named-setup-rndc.service
/usr/lib/systemd/system/named.service
```
bind程序安装完成之后，默认即可做缓存名称服务器使用；如果没有专门负责解析的区域，直接即可启动服务。
- CentOS 6: `service  named  start`
- CentOS 7: `systemctl  start  named.service`

named 进程启动后，默认会监听 tcp 的 953 号端口，rndc 可通过此端口对 named 进程进行远程控制。但是 named 进程默认只监听在 127.0.0.1 上，因此仅允许本地使用。

#### rndc
rndc：remote name domain contoller
```
rndc [-b address] [-c config] [-s server] [-p port]
	[-k key-file ] [-y key] [-V] command

command is one of the following:

    reload  Reload configuration file and zones.
    reload zone [class [view]]  Reload a single zone.
    refresh zone [class [view]]   Schedule immediate maintenance for a zone.
    stats     Write server statistics to the statistics file.
    status   Display status of the server.
    stop       Save pending updates to master files and stop the server.
    stop -p   Save pending updates to master files and stop the server
    flush     Flushes all of the server's caches.
    flush [view]  Flushes the server's cache for a view.
```

## 2. bind 配置
### 2.1 配置格式
```bash
$ cat /etc/named.conf
options {                                 # 全局配置段
	listen-on port 53 { 127.0.0.1; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";               # 区域解析库文件的默认存放目录
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { localhost; };

	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {                                # 日志配置段
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {                            # 顶级域解析库配置文件
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";     # 包含的辅助配置文件
include "/etc/named.root.key";
```
bind 的主配置 `/etc/named.conf` 由三个部分组成:
1. 全局配置段：`options { ... }`
2. 日志配置段: `logging { ... }`
3. 区域配置段: `zone { ... }` 配置那些由本机负责解析的区域，或转发的区域

`/etc/named.conf`有如下语法要求:
1. 每个配置语句必须以分号结尾
2. `{}` 左右都必需要有空格
3. 使用 `//` 或 `/* */` 进行注释

### 2.2 缓存名 DNS 服务器配置
named 进程默认启动后，即可作为缓存 DNS 服务器，但是默认配置只允许本地查询，无法对外提供服务，因此需要做如下修改。
1. 更改 named 监听的地址，使其能与外部主机通信的地址；
2. 学习使用时，建议关闭 dnssec
3. 关闭仅允许本地查询配置

配置文件修改后应该使用`named-checkconf`，检查配置文件语法是否存在错误，在重起 named 进程。下面是配置过程

```bash
$ vim /etc/named.conf
	# 修改监听地址，使其能与外部主机通信的地址；					
	listen-on port 53;
	listen-on port 53 { 172.16.100.67; };

	# 学习时，建议关闭dnssec
	dnssec-enable no;
	dnssec-validation no;
	dnssec-lookaside no;

	# 关闭仅允许本地查询：
	//allow-query  { localhost; };

# 检查配置文件语法
$ named-checkconf
```


## 3. 正反向解析区域配置
上一节我们学习过区域解析库文件的语法格式，现在我们就来学习，如何配置正向和反向区域。我们将以配置`magedu.com` 这个二级域为例。

### 3.1 正向区域配置
配置一个正向解析区域需要:
1. 定义区域: 在主配置文件中或主配置文件辅助配置文件中定义区域，**区域名字即为域名**
2. 建立区域数据文件，正向区域的主要记录为A或AAAA记录，并更改配置文件属性
3. 重启服务: 检查语法错误，然后让服务器重载配置文件和区域数据文件

下面是配置的详细过程:
#### 定义区域
```bash
$ vim /etc/named.rfc1912.zones  # 追加
zone  "magedu.com."  IN  {
	type  master;   
	file  "magedu.com.zone";
};
```
区域定义参数:
- `type`: 用于定义区域的主机类型，可选值为
	- `master`: 主服务器
	- `slave`: 从服务器
	- `hint`: 根域
	- `forward`
- `file`: 指定区域数据库文件位置，使用相对路经，则在 `/etc/named.conf` 配置的默认路经`/var/named`之下

#### 建立区域数据文件
```bash
$ cd /var/named          
$ touch /var/named/magedu.com.zone          # 创建区域数据文件
$ chgrp  named  /var/named/magedu.com.zone  # 更改权限及属组
$ chmod  o=  /var/named/magedu.com.zone

$ vim /var/named/magedu.com.zone
$TTL 3600
$ORIGIN magedu.com.        #  ORIGIN 会自动补全下面 ns，mx1 的域名
@	   IN	  SOA	 ns1.magedu.com.   dnsadmin.magedu.com. (
							2017010801
							1H
							10M
							3D
							1D)
	    IN	  NS	    ns1
      IN	  MX   10 mx1
      IN	  MX   20 mx2
ns1	  IN	  A	   172.16.100.67
mx1	  IN	  A	   172.16.100.68
mx2	  IN	  A	   172.16.100.69
www	  IN	  A	   172.16.100.67
web	  IN	  CNAME   www
bbs	  IN	  A	   172.16.100.70
bbs	  IN	  A	   172.16.100.71
```

#### 重启服务
```bash
# 检查语法错误
$ named-checkzone magedu.com.  /var/named/magedu.com.zone
$ named-checkconf

# 重起服务
$ rndc  reload   # 或
$ systemctl  reload  named.service
```

### 3.2 反向解析区域配置
配置反向区域与配置正向区域类似，只不过**区域名称**和**区域数据库文件**不同:
1. 反向区域的名字格式为: `反写的网段地址.in-addr.arpa`, 例如区域 `172.16.100` 的区域名称为 `100.16.172.in-addr.arpa`
2. 反向区域没有 `MX` 资源记录，主要为 `PTR` 资源记录

#### 定义区域
```bash
$ vim /etc/named.rfc1912.zones  # 追加
zone  "100.16.172.in-addr.arpa"  IN  {
	type  master;   
	file  "172.16.100.zone";
};
```

#### 建立区域数据文件
```bash
$ cd /var/named          
$ touch /var/named/172.16.100.zone          # 创建区域数据文件
$ chgrp  named  /var/named/172.16.100.zone  # 更改权限及属组
$ chmod  o=  /var/named/172.16.100.zone

$ vim /var/named/172.16.100.zone
$TTL 3600
$ORIGIN 100.16.172.in-addr.arpa.
@	   IN	  SOA	 ns1.magedu.com.  nsadmin.magedu.com. (
						  2017010801
						  1H
						  10M
						  3D
						  12H )
		  IN	  NS	 ns1.magedu.com.      
67	  IN	  PTR	 ns1.magedu.com.   # 域名必需写全，此时 ORIGIN 不能补全
68	  IN	  PTR	 mx1.magedu.com.
69	  IN	  PTR	 mx2.magedu.com.
70	  IN	  PTR	 bbs.magedu.com.
71	  IN	  PTR	 bbs.magedu.com.
67	  IN	  PTR	 www.magedu.com.					
```

#### 重启服务
```bash
# 检查语法错误
$ named-checkzone 100.16.172.in-addr.arpa  /var/named/172.16.100.zone
$ named-checkconf

# 重起服务
$ rndc  reload   # 或
$ systemctl  reload  named.service
```


## 4. DNS 测试工具
#### dig
`dig  [-t RR_TYPE]  name  [@SERVER]  [query options]`
- 作用: 用于测试dns系统，因此其不会查询hosts文件；
- 参数:
	- `name`: DNS 资源记录的名称
	- `@SERVER`: 可选，使用的 DNS 服务器，默认为本机
- 选项：
	- `-t RR_TYPE`: 指定查询的资源记录类型
	- `+[no]trace`：跟踪解析过程
	- `+[no]recurse`：进行递归解析；
	- `-x IP`:进行反向解析测试
- 常用:
	- 反向解析测试 : `dig -x IP`
	- 模拟完全区域传送： `dig -t axfr DOMAIN [@server]`

```bash
# 正向解析测试
$  dig -t NS baidu.com.
$  dig -t A  www.baidu.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> -t A www.baidu.com +recurse
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36303
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; ANSWER SECTION:
www.baidu.com.		126	IN	CNAME	www.a.shifen.com.
www.a.shifen.com.	126	IN	A	61.135.169.125
www.a.shifen.com.	126	IN	A	61.135.169.121

;; Query time: 47 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: 二 8月 14 11:54:53 CST 2018
;; MSG SIZE  rcvd: 90

# 反向解析测试
$ dig  -x  61.135.169.125
```

#### host
`host  [-t  RR_TYPE]  name  [SERVER_IP]`
- 作用: 用于测试dns系统，不会查询hosts文件；
- 参数:
	- `name`: DNS 资源记录的名称
	- `SERVER_IP`: 可选，使用的 DNS 服务器，默认为本机
- 选项：
	- `-t RR_TYPE`: 指定查询的资源记录类型

```
$ host -t A www.baidu.com
www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 61.135.169.125
www.a.shifen.com has address 61.135.169.121

$ host -t PTR 61.135.169.121
Host 121.169.135.61.in-addr.arpa. not found: 3(NXDOMAIN)
```

#### nslookup
`nslookup  [-options]  [name]  [server]`
- 作用: 用于测试dns系统，默认进入交互式环境

```
nslookup>
server  IP      # 指定DNS服务器
set  q=RR_TYPE  # 要查询的资源记录类型；
name           # 要查询的名称


$ nslookup
> set q=A
> www.baidu.com
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
www.baidu.com	canonical name = www.a.shifen.com.
Name:	www.a.shifen.com
Address: 61.135.169.121
Name:	www.a.shifen.com
Address: 61.135.169.125
>
```			
