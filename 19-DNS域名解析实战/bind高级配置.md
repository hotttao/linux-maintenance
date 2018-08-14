# 19.4 bind 高级配置
本节是 bind 配置的高级篇，实现
1. 主从 DNS 服务器配置
2. DNS  子域授权
3. 定义 DNS 的转发
4. DNS 访问控制
5. 智能 DNS

## 1. 主从服务器
DNS 的主从配置，是以域名解析的区域为基本单位的，也就是说如果一台主机上配置了多个 DNS 解析域，为哪个区域配置了从域，哪个区域就实现了主从服务器配置。

配置从区域首先要在从服务器上配置 DNS 服务，但配置方法更简单，因为只要定义一个区域即可，无需配置区域解析库文件。然后在主 DNS 的解析库文件中添加从服务器。需要特别注意的是主从服务器的时间要同步否则无法实现主从同步，可使用 `ntpdate`命令完成时间同步。具体步骤如下

### 1.1 On Slave
从域配置:
1. 定义区域: 定义一个从区域；
2. 配置文件语法检查：`named-checkconf`
3. 重载配置

```bash
$ vim /etc/named.rfc1912.zones  # 追加
zone "ZONE_NAME"  IN {
    type  slave;
    file  "slaves/ZONE_NAME.zone"; # /var/named 目录是不允许 named 用户写的
    masters  { MASTER_IP; };       # 主服务器的 ip 地址
  };

$ named-checkconf

$ rndc  reload
```

### 1.2 On Master
主域配置:
1. 确保区域数据文件中为每个从服务配置NS记录，
2. 并且在**正向区域文件**需要每个从服务器的NS记录的主机名配置一个A记录，且此A后面的地址为真正的从服务器的IP地址；

```bash
$ vim /var/named/ZONE_NAME.zone
@    IN   NS    ns2     # 从服务器具的 NS 记录

$ vim /var/named/ZONE_FQDN.zone  # 必需在正向解析库文件中添加 A 记录
@    IN   NS    ns2     # 从服务器具的 NS 记录
ns2  IN   A     ip_addr # 从服务器的 ip 地址
```

## 2. 子域授权
正向解析区域授权子域的方法，以在 `magedu.com.` 的二级域中授权 `ops.magedu.com.` 三级域为例:

```bash
$ vim /var/named/ZONE_NAME.zone  # 二级域的区域数据文件
ops.magedu.com. 		IN 	NS  	ns1.ops.magedu.com.   # 子域的 NS 记录
ops.magedu.com. 		IN 	NS  	ns2.ops.magedu.com.
ns1.ops.magedu.com. 	IN 	A 	IP.AD.DR.ESS          # 子域的 A 记录
ns2.ops.magedu.com. 	IN 	A 	IP.AD.DR.ESS
```

## 3. 定义转发
默认情况下，DNS 服务在解析非自己负责的域名时，默认会向根域发起迭代查询。我们可以定义转发域，让 DNS 服务在解析非自己负责的域名时，向被转发服务器发起第归查询而不是向根域迭代查询。因此被转发的服务器必须允许为当前服务做递归。转发可分为区域转发和全局转发:

### 3.1 区域转发
区域转发: 仅转发对某特定区域的解析请求；配置方法如下

```bash
$ vim /etc/named.rfc1912.zones  # 追加
zone  "ZONE_NAME"  IN {
  type  forward;
  forward  {first|only};
  forwarders  { SERVER_IP; };
  };
```
参数说明:
- `forward`:定义转发的特性
	- `first`：首先转发；转发器不响应时，自行去迭代查询；
	- `only`：只转发；
- `forwarders`: 被转发服务器的 IP

### 3.2 全局转发
全局转发：凡本地没有通过zone定义的区域查询请求，通通转给某转发器；

```bash
$ vim /etc/named.conf  # 追加
options {
  ... ...
  forward  {only|first};
  forwarders  { SERVER_IP; };
  .. ...
};
```

## 4. bind 安全配置
bind 的配置文件中可以使用 acl(访问控制列表)，把一个或多个地址归并一个命名的集合，随后通过此名称即可对此集全内的所有主机实现统一调用；

acl 只能先定义后使用，所以通常位于 `/etc/named.conf` 最上面，并且是独立的配置段。
### 4.1 acl

```
acl  acl_name  {
       ip;
       net/prelen;
};

# eg:
acl  mynet {
	172.16.0.0/16;
	127.0.0.0/8;
};
```

### 4.2 bind内置的acl
bind 由四个内置的 acl:
- `none`：没有一个主机；
- `any`：任意主机；
- `local`：本机；
- `localnet`：本机所在的IP所属的网络；

### 4.3 访问控制指令
访问控制指令位于 `options` 表示对全局生效，位于 `zone`区域段中，表示只对此区域有效，常见的控制指令有
- `allow-query  {}`  
    - 允许查询的主机；白名单，未在此范围内的不能发起查询
- `allow-transfer {}`  
    - 允许向哪些主机做区域传送；默认为向所有主机；
    - 应该配置仅允许从服务器；如果没有从服务器，必需设置为 None
- `allow-recursion {}`
    - 允许哪些主机向当前DNS服务器发起递归查询请求；
- `allow-update {}`
    - DDNS，允许动态更新区域数据库文件中内容；存在风险，一般都为 none

```bash
$ vim /etc/named.conf
acl "slaves" {
    172.168.100.68;
    172.168.0.0/16;
}

$ vim /etc/named.rfc1912.zones  # 追加
zone "magedu.com." IN {
    type master;
    file "magedu.com.zone"
    allow-transfer { slaves; };
    allow-update { none; };
}
```

## 5. 智能 DNS - bind view
视图主要作用是实现，来自不同的用户的请求，可以返回不同的地址。比如来自内网的用户，得到的是内网的地址，来自公网的用户得到的是公网地址。
```
	view  VIEW_NAME {
	  zone
	  zone
	  zone
	}
```

#### 示例
```
view internal  {                   # 优先匹配的位于上面
  match-clients { 172.16.0.0/8; };
  zone "magedu.com"  IN {
	   type master;
	   file  "magedu.com/internal"; # 内网的解析库文件
  };
};


view external {
  match-clients { any; };
  zone "magecdu.com" IN {
	   type master;
	   file magedu.com/external"; # 外网的解析库文件
  };
};
```

#### whois
