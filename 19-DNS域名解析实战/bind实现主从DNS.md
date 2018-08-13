# 19.4 基于bind实现主从、智能DNS

## 3. 主从服务器：
 注意：从服务器是区域级别的概念；

### 3.1 配置一个从区域：
**On Slave**
1. 定义区域: 定义一个从区域；
	```
	  zone "ZONE_NAME"  IN {
		type  slave;
		file  "slaves/ZONE_NAME.zone";
		masters  { MASTER_IP; };
	  };
	```
2. 配置文件语法检查：named-checkconf
3. 重载配置
	- rndc  reload
	- systemctl  reload  named.service

**On Master**
1. 确保区域数据文件中为每个从服务配置NS记录，
2. 并且在正向区域文件需要每个从服务器的NS记录的主机名配置一个A记录，
3. 且此A后面的地址为真正的从服务器的IP地址；
4. 注意：时间要同步；ntpdate命令；

### 3.2 子域授权：
正向解析区域授权子域的方法：
```
ops.magedu.com. 		IN 	NS  	ns1.ops.magedu.com.
ops.magedu.com. 		IN 	NS  	ns2.ops.magedu.com.
ns1.ops.magedu.com. 	IN 	A 	IP.AD.DR.ESS
ns2.ops.magedu.com. 	IN 	A 	IP.AD.DR.ESS
```

### 3.3 定义转发：
注意：被转发的服务器必须允许为当前服务做递归；
1. 区域转发：仅转发对某特定区域的解析请求；
	- first：首先转发；转发器不响应时，自行去迭代查询；
	- only：只转发；
	```
	  zone  "ZONE_NAME"  IN {
		type  forward;
		forward  {first|only};
		forwarders  { SERVER_IP; };
	  };
	```
2. 全局转发：针对凡本地没有通过zone定义的区域查询请求，通通转给某转发器；
	```
	 options {
		... ...
		forward  {only|first};
		forwarders  { SERVER_IP; };
		.. ...
	 };
	```

## 4. bind中的安全相关的配置
### 4.1 acl：
- 访问控制列表；
- 把一个或多个地址归并一个命名的集合，随后通过此名称即可对此集全内的所有主机实现统一调用；

```
acl  acl_name  {
       ip;
       net/prelen;
};

示例：
acl  mynet {
	172.16.0.0/16;
	127.0.0.0/8;
};
```

### 4.2 bind四个内置的acl
- none：没有一个主机；
- any：任意主机；
- local：本机；
- localnet：本机所在的IP所属的网络；

### 4.3 访问控制指令：
- allow-query  {};  允许查询的主机；白名单；
- allow-transfer {};  允许向哪些主机做区域传送；默认为向所有主机；应该配置仅允许从服务器；
- allow-recursion {}; 允许哪此主机向当前DNS服务器发起递归查询请求；
- allow-update {}; DDNS，允许动态更新区域数据库文件中内容；

## 5. bind view：
视图：
```
	view  VIEW_NAME {
	  zone
	  zone
	  zone
	}
```

示例
```
	view internal  {
	  match-clients { 172.16.0.0/8; };
	  zone "magedu.com"  IN {
		type master;
		file  "magedu.com/internal";
	  };
	};

	view external {
	  match-clients { any; };
	  zone "magecdu.com" IN {
		type master;
		file magedu.com/external";
	  };
	};
```
