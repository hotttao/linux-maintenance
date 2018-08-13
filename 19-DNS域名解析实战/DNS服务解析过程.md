# 19.2 DNS服务解析过程精讲
### 1.3 DNS 服务器
#### 1.3.1 主-辅DNS服务器：
- 主DNS服务器：维护所负责解析的域数据库的那台服务器；读写操作均可进行；
- 从DNS服务器：从主DNS服务器那里或其它的从DNS服务器那里“复制”一份解析库；但只能进行读操作；
- 主从同步方式:
	- “复制”操作的实施方式：
		- 序列号：serial, 也即是数据库的版本号；主服务器数据库内容发生变化时，其版本号递增；
		- 刷新时间间隔：refresh, 从服务器每多久到主服务器检查序列号更新状况；
		- 重试时间间隔：retry, 从服务器从主服务器请求同步解析库失败时，再次发起尝试请求的时间间隔；
		- 过期时长：expire，从服务器始终联系不到主服务器时，多久之后放弃从主服务器同步数据；停止提供服务；
		- 否定答案的缓存时长：
	- 主服务器”通知“从服务器随时更新数据；
- 区域传送：
	- 全量传送：axfr, 传送整个数据库；
	- 增量传送：ixfr, 仅传送变量的数据；
- 区域(zone)和域(domain)：
	- magedu.com域：FQDN（Full Qualified Domain Name）
		- FQDN --> IP: 正向解析库；区域
		- IP --> FQDN: 反向解析库；区域

#### 1.3.2 区域数据库文件：
资源记录：Resource Record, 简称rr；
- 记录有类型：A， AAAA， PTR， SOA， NS， CNAME， MX
	- SOA：Start Of Authority，起始授权记录； 一个区域解析库有且只能有一个SOA记录，而且必须放在第一条；
	- NS：Name Service，域名服务记录；一个区域解析库可以有多个NS记录；其中一个为主的；
	- A： Address, 地址记录，FQDN --> IPv4；
	- AAAA：地址记录， FQDN --> IPv6；
	- CNAME：Canonical Name，别名记录；
	- PTR：Pointer，IP --> FQDN
	- MX：Mail eXchanger，邮件交换器；优先级：0-99，数字越小优先级越高；

### 1.4 DNS 资源记录的定义格式：
- 语法：	name  	[TTL] 	IN	RR_TYPE 		value
- 注意：
	- TTL可以从全局继承；
	- @表示当前区域的名称；
	- 相邻的两条记录其name相同时，后面的可省略；
	- 对于正向区域来说，各MX，NS等类型的记录的value为FQDN，此FQDN应该有一个A记录；
- SOA：
	- name: 当前区域的名字；例如”mageud.com.”，或者“2.3.4.in-addr.arpa.”；
	- value：有多部分组成
		- 当前区域的区域名称（也可以使用主DNS服务器名称）；
		- 当前区域管理员的邮箱地址；但地址中不能使用@符号，一般使用点号来替代；
		- (主从服务协调属性的定义以及否定答案的TTL)
	```
	例如：
	  magedu.com. 	86400 	IN 		SOA 	magedu.com. 	admin.magedu.com.  (
		2017010801	; serial
		2H 			; refresh
		10M 		; retry
		1W			; expire
		1D			; negative answer ttl
	  )
	```
- NS：
 	- name: 当前区域的区域名称
 	- value：当前区域的某DNS服务器的名字，例如ns.magedu.com.；
	- 注意：一个区域可以有多个ns记录；
	```
	例如：
	magedu.com. 	86400 	IN 	NS  	ns1.magedu.com.
	magedu.com. 	86400 	IN 	NS  	ns2.magedu.com.
	```
- MX：
	- name: 当前区域的区域名称
	- value：当前区域某邮件交换器的主机名；
	- 注意：MX记录可以有多个；但每个记录的value之前应该有一个数字表示其优先级；
	```
	例如：
	magedu.com. 		IN 	MX 	10  	mx1.magedu.com.
	magedu.com. 		IN 	MX 	20  	mx2.magedu.com.
	```

- A：
	- name：某FQDN，例如www.magedu.com.
	- value：某IPv4地址；
	```
	例如：
	www.magedu.com.		IN 	A	1.1.1.1
	www.magedu.com.		IN 	A	1.1.1.2
	bbs.magedu.com.			IN 	A	1.1.1.1
	```
- AAAA：
	- name：FQDN
	- value: IPv6
- PTR：
	- name：IP地址，有特定格式，IP反过来写，而且加特定后缀；例如1.2.3.4的记录应该写为4.3.2.1.in-addr.arpa.；
	- value：FQND
	- 例如：4.3.2.1.in-addr.arpa.  	IN  PTR	www.magedu.com.
- CNAME：
	- name：FQDN格式的别名；
	- value：FQDN格式的正式名字；
	- 例如: web.magedu.com.  	IN  	CNAME  www.magedu.com.
