# 19.3 基于Linux平台bind安装和配置



## 1. BIND
### 1.1 bind 的安装
BIND： Berkeley Internet Name Domain,  ISC.org
- 对应关系：
	- dns: 协议
	- bind： dns协议的一种实现
	- named：bind程序的运行的进程名
- 程序包：
	- bind-libs：被bind和bind-utils包中的程序共同用到的库文件；
	- bind-utils：bind客户端程序集，例如dig, host, nslookup等；
	- bind：提供的dns server程序、以及几个常用的测试程序；
	- bind-chroot：选装，让named运行于jail模式下；

## 1.2 bind 配置文件
- 主配置文件：/etc/named.conf
- 或包含进来其它文件；
	- /etc/named.iscdlv.key
	- /etc/named.rfc1912.zones
	- /etc/named.root.key
- 解析库文件：
	- /var/named/目录下；
	- 一般名字为：ZONE_NAME.zone
- 注意：
	- 一台DNS服务器可同时为多个区域提供解析；
	- 必须要有根区域解析库文件： named.ca；
	- 还应该有两个区域解析库文件：localhost和127.0.0.1的正反向解析库；
		- 正向：named.localhost
		- 反向：named.loopback

### 1.3 bind 启动
1. rndc：remote name domain contoller
	- 953/tcp，但默认监听于127.0.0.1地址，因此仅允许本地使用；
2. bind程序安装完成之后，默认即可做缓存名称服务器使用；如果没有专门负责解析的区域，直接即可启动服务；
	- CentOS 6: service  named  start
	- CentOS 7: systemctl  start  named.service

## 2. bind 配置
### 2.1 配置格式
1. 主配置文件格式：
	- 全局配置段：
	- options { ... }
2. 日志配置段：
	- logging { ... }
3. 区域配置段：
	- zone { ... }
	- 那些由本机负责解析的区域，或转发的区域；
	- 注意：每个配置语句必须以分号结尾；

### 2.2 缓存名称服务器的配置：
- 监听能与外部主机通信的地址；					
	- listen-on port 53;
	- listen-on port 53 { 172.16.100.67; };
- 学习时，建议关闭dnssec
	- dnssec-enable no;
	- dnssec-validation no;
	- dnssec-lookaside no;
- 关闭仅允许本地查询：
	- //allow-query  { localhost; };
- 检查配置文件语法错误：
	- named-checkconf   [/etc/named.conf]

### 2.3  测试工具：
**dig  [-t RR_TYPE]  name  [@SERVER]  [query options]**
- 作用: 用于测试dns系统，因此其不会查询hosts文件；
- 查询选项：
	- +[no]trace：跟踪解析过程；
	- +[no]recurse：进行递归解析；
- 反向解析测试 :    dig  -x  IP
- 模拟完全区域传送： dig  -t  axfr  DOMAIN  [@server]

**host  [-t  RR_TYPE]  name  SERVER_IP**

**nslookup  [-options]  [name]  [server]**
```
交互式模式：
nslookup>
server  IP：以指定的IP为DNS服务器进行查询；
set  q=RR_TYPE：要查询的资源记录类型；
name：要查询的名称；
```

**rndc命令**：
- named服务控制命令
- rndc  status
- rndc  flush

### 2.4 配置解析一个正向区域：
以magedu.com域为例：
1. 定义区域: 在主配置文件中或主配置文件辅助配置文件中实现；
	- 注意：区域名字即为域名；
	```
	  zone  "ZONE_NAME"  IN  {
		type  {master|slave|hint|forward};
		file  "ZONE_NAME.zone";
	  };
	```
2. 建立区域数据文件（主要记录为A或AAAA记录）
	- 在/var/named目录下建立区域数据文件；
	- 文件为：/var/named/magedu.com.zone
	- 权限及属组修改：
		- chgrp  named  /var/named/magedu.com.zone
		- chmod  o=  /var/named/magedu.com.zone
	- 检查语法错误：
		- named-checkzone  ZONE_NAME   ZONE_FILE
		- named-checkconf
	```
	$TTL 3600
	$ORIGIN magedu.com.
	@	   IN	  SOA	 ns1.magedu.com.   dnsadmin.magedu.com. (
		2017010801
		1H
		10M
		3D
		1D )
	  IN	  NS	  ns1
	  IN	  MX   10 mx1
	  IN	  MX   20 mx2
	ns1	 IN	  A	   172.16.100.67
	mx1	 IN	  A	   172.16.100.68
	mx2	 IN	  A	   172.16.100.69
	www	 IN	  A	   172.16.100.67
	web	 IN	  CNAME   www
	bbs	 IN	  A	   172.16.100.70
	bbs	 IN	  A	   172.16.100.71
	```
3. 让服务器重载配置文件和区域数据文件
	- rndc  reload 或
	- systemctl  reload  named.service

### 2.3 配置解析一个反向区域
1. 定义区域: 在主配置文件中或主配置文件辅助配置文件中实现；
	- 注意：反向区域的名字, 反写的网段地址.in-addr.arpa,  100.16.172.in-addr.arpa
	 ```
	 zone  "ZONE_NAME"  IN  {
		type  {master|slave|hint|forward};
		file  "ZONE_NAME.zone";
	 };
	 ```
2. 定义区域解析库文件（主要记录为PTR）
	- 权限及属组修改：
		- chgrp  named  /var/named/172.16.100.zone
		- chmod  o=  /var/named/172.16.100.zone
	- 检查语法错误：
		- named-checkzone  ZONE_NAME   ZONE_FILE
		- named-checkconf
	- 示例，区域名称为100.16.172.in-addr.arpa；
	```
	  $TTL 3600
	  $ORIGIN 100.16.172.in-addr.arpa.
	  @	   IN	  SOA	 ns1.magedu.com.  nsadmin.magedu.com. (
		  2017010801
		  1H
		  10M
		  3D
		  12H )
		IN	  NS	  ns1.magedu.com.
	  67	  IN	  PTR	 ns1.magedu.com.
	  68	  IN	  PTR	 mx1.magedu.com.
	  69	  IN	  PTR	 mx2.magedu.com.
	  70	  IN	  PTR	 bbs.magedu.com.
	  71	  IN	  PTR	 bbs.magedu.com.
	  67	  IN	  PTR	 www.magedu.com.					
	 ```
3. 让服务器重载配置文件和区域数据文件
	- rndc  reload 或
	- systemctl  reload  named.service					
