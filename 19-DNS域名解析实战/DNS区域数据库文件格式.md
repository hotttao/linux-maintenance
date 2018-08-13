# 19.2 DNS区域数据库文件格式
DNS 的数据库文件记录了 FQDN 与 IP 的对应关系，有特定格式要求。本节我们就来学习DNS区域数据库文件格式

## 1. 资源类型
DNS区域数据库文件一条记录为一行，被称为**资源记录**(Resource Record), 简称`rr`。常见的资源记录类型包括:
- `SOA`：Start Of Authority，起始授权记录； 一个区域解析库有且只能有一个SOA记录，而且必须放在第一条；
- `NS`：Name Service，域名服务记录；一个区域解析库可以有多个NS记录；其中一个为主的；
- `A`： Address, 地址记录，`FQDN --> IPv4`；
- `AAAA`：地址记录， `FQDN --> IPv6`；
- `CNAME`：Canonical Name，别名记录；
- `PTR`：Pointer，`IP --> FQDN`
- `MX`：Mail eXchanger，邮件交换器；优先级：0-99，数字越小优先级越高

每种资源记录有特定的格式要求，接下来我们来分别介绍

## 2. DNS 资源记录的定义格式
`name  	[TTL] 	IN	RR_TYPE 		value`
- 作用: DNS 资源记录定义的语法
- 参数:
	- `name`: 名称
	- `value`: 名称对应的值与属性
	- `TTL`: Time To Live,有效是长
	- `IN`: 关键字
	- `RR_TYPE`： 资源类型
- 注意：
	- TTL可以从全局继承；
	- `@`表示当前区域的名称；
	- 相邻的两条记录其name相同时，后面的可省略；
	- 对于正向区域来说，各MX，NS等类型的记录的value为FQDN，此FQDN应该有一个A记录；
	- 配置文件内 `;` 后跟注释
	- FQDN 最后的根域名`.`不可省略

下面是各个资源记录的配置示例

```
# SOA
leistudy.com.   86400   IN  SOA     ns.leistudy.com.    nsadmin.leistudy.com.   (
            2018022801  ;序列号
            2H          ;刷新时间
            10M         ;重试时间
            1W          ;过期时间
            1D          ;否定答案的TTL值
)

# NS
leistudy.com.   IN  NS  ns1.leistudy.com.
leistudy.com.   IN  NS  ns2.leistudy.com.

# MX
leistudy.com.   IN  MX  10  mx1.leistudy.com.
                IN  MX  20  mx2.leistudy.com.

# A
www.leistudy.com.   IN  A   1.1.1.1
www.leistudy.com.   IN  A   1.1.1.2
mx1.leistudy.com.   IN  A   1.1.1.3
mx2.leistudy.com.   IN  A   1.1.1.3

# AAAA
www.leistudy.com.   IN  AAAA   ::1

# PTR
4.3.2.1.in-addr.arpa. IN   PTR www.leistudy.com
```

### 2.1 SOA
`name  	[TTL] 	IN	RR_TYPE  value`
- `name`: 当前区域的名字；例如`mageud.com.`，或者`2.3.4.in-addr.arpa.`；
- `value`：有多部分组成
	1. 当前区域的区域名称（也可以使用主DNS服务器名称）；
	2. 当前区域管理员的邮箱地址；但地址中不能使用`@`符号，一般使用点号来替代
	3. (主从服务协调属性的定义以及否定答案的TTL)

```
magedu.com. 	86400 	IN 		SOA 	magedu.com. 	admin.magedu.com.  (
		2017010801	; serial             主从服务协调属性
		2H 					; refresh
		10M 				; retry
		1W					; expire
		1D					; negative answer ttl 否定答案的 TTL
)
```

### 2.2 NS
`name  	[TTL] 	IN	RR_TYPE 		value`
- `name`: 当前区域的区域名称
- `value`: 当前区域的某DNS服务器的名字，例如ns.magedu.com.；
- 注意：
	- 一个区域可以有多个ns记录；
	- 相邻的两条记录其name相同时，后面的可省略；

```
magedu.com.   86400 	IN 	NS  	ns1.magedu.com.
              86400 	IN 	NS  	ns2.magedu.com.
```

### 2.3 MX
`name  	[TTL] 	IN	RR_TYPE 		value`
- `name`: 当前区域的区域名称
- `value`：当前区域某邮件交换器的主机名；
- 注意：MX记录可以有多个；但每个记录的value之前应该有一个数字表示其优先级；

```
# @ 可表示当前区域的名称
@ 		IN 	MX 	10  	mx1.magedu.com.
@ 		IN 	MX 	20  	mx2.magedu.com.
```

### 2.4  A
`name  	[TTL] 	IN	RR_TYPE 		value`
- `name`：某FQDN，例如www.magedu.com.
- `value`：某IPv4地址；

```
www.magedu.com.		IN 	A	1.1.1.1
www.magedu.com.		IN 	A	1.1.1.2
bbs.magedu.com.		IN 	A	1.1.1.1
```

### 2.5 AAAA
`name  	[TTL] 	IN	RR_TYPE 		value`
- `name`：FQDN
- `value`: IPv6

```
www.magedu.com. IN AAAA ::1
```

### 2.6 PTR
`name  	[TTL] 	IN	RR_TYPE 		value`
- 作用: 反向解析的资源记录格式
- `name`：IP地址，IP必需反过来写，而且必需加特定后缀；例如 `1.2.3.4` 的记录应该写为`4.3.2.1.in-addr.arpa.`
- `value`：FQND

```
4.3.2.1.in-addr.arpa.  	IN  PTR	 www.magedu.com.
```

### 2.7 CNAME
`name  	[TTL] 	IN	RR_TYPE 		value`
- `name`: FQDN格式的别名；
- `value`: FQDN格式的正式名字；

```
web.magedu.com.  	IN  	CNAME  www.magedu.com.
```
