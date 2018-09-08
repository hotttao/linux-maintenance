# 27.6 防火墙标记和LVS持久链接
前面我们演示了如何使用 LVS 创建一个负载均衡服务，然而在生产环境中，我们可能要同时调度两个及以上的集群服务。典型的情景是，同时部署了 http，https 服务，用户浏览网页的时候使用的 http 服务，当用户登陆或支付时因为 http 是明文的不安全，此时必需切换成 https。如果这个两个服务是单独调度，很有可能用户登陆之后被重新调度到其他服务器上，这样用户原有的缓存就会丢失。所以我们必需将 http，https 作为一组服务进行同一调度，这就需要使用到防火墙标记。我们本节我们就来演示如何使用LVS 统一调度 http，https 服务。

## 1. 防火墙标记 FWM
要想将一组RS的集群服务统一进行调度，我们需要借助 iptables 的防火墙标记功能(FWM)
1. 首先在 director `iptables` 的 `mangle` 表的 `PREROUTING` 链上对一组服务标打上同样的防火墙标记
2. 然后基于FWM 定义集群服务，让 ipvs 对相同标记的服务进行统一调度

## 2. http/https 统一调度示例
上一节我们基于 LVS-DR 配置了一个 http 服务，在此基础上我们继续配置一个 https 服务，并将它们统一调度

### 2.1 RS https 服务
https 的负载均衡
1. 首先各个 RS 密钥和证书文件必需一致
2. ssh 回话会大量的耗费系统资源，因此服务器会对 ssh 会话进行缓存，为了使缓存生效，lvs 必需使用 sh 算法进行调度，但是 sh 算法会影响负载均衡的效果
3. 如果负载均衡器是 nginx 我们可以在调度器上进行 ssh 会话的建立和缓存，发送到后端服务器请求就可以直接基于 http 协议。我们称这种方式为 ssh 会话拆除。但是  LVS 是工作在内核上，无法理解 ssh 会话，也就做不到 ssh 会话拆除。

```bash
# 1. VS 上私建 CA
# CA
(umask 077;openssl genrsa -out private/cakey.pem 2048)
openssl req -new -x509 -key private/cakey.pem -out cacert.pem -days 365

# 证书
(umask 077;openssl genrsa -out /root/http.key 2048)
openssl req -new -key /root/http.key -out /root/http.csr -days 365
openssl ca -in /root/http.csr -out /root/http.crt -days 365

# 2. RS 的 https 服务配置
scp /root/http.* root@192.168.1.107:/etc/httpd/ssl
yum install mod_ssl
vim /etc/httpd/cond/ssl.conf

# 3.验证 https 服务
# 在客户端导入证书
scp /etc/pki/CA/cacert.pem root@192.168.1.106:/root/
vim /etc/hosts # 添加域名
curl --cacert /root/cacert.pem  "https://www.tao.com/test.html"
```
### 2.2 VS 集群服务配置

```bash
case $1 in
start)
	iptables -F
	ipvsadm -C

	iptables -t mangle -A PREROUTING -d 192.168.1.99 -p tcp -m multiport --dports 443,80 -j MARK --set-mark 3

	ipvsadm -A -f 3 -s sh
	ipvsadm -a -f 3 -r 192.168.1.107 -g -w 1
	ipvsadm -a -f 3 -r 192.168.1.109 -g -w 1
	;;
stop)
	iptables -F
	ipvsadm -C
	;;
esac
```

### 2.3 测试
```bash
for i in {1..10};do curl --cacert /root/cacert.pem  "https://www.tao.com/test.html";curl --cacert /root/cacert.pem  "http://www.tao.com/test.html";done
```

## 3. lvs persistence
lvs 持久连接
- 功能: 实现无论使用任何调度算法，在一段时间内，能够实现将来自同一个地址的请求始终发往同一个RS；
- 实现: lvs 的持久连接模板，独立于调度算法存在
- 类型:
    - `PPC`: 每个端口对应定义为一个集群服务，每集群服务单独调度；
    - `PFWMC`: 基于防火墙标记定义集群服务；可实现将多个端口上的应用统一调度，即所谓的port Affinity；
    - `PCC`: 基于0端口定义集群服务，即将客户端对所有应用的请求统统调度至后端主机，必须定义为持久模式；
- 启用: `ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]]`
    - `-p [timeout]`: 使用 `-p` 参数就可以启用 lvs 持久链接功能，默认为 360 秒

```bash
# PPC:
ipvsadm -A -t 192.168.1.99:80  -s rr -p [600]
ipvsadm -a -t 192.168.1.99:80  -r 192.168.1.107 -g
ipvsadm -a -t 192.168.1.99:80  -r 192.168.1.109 -g

# PFWMC -- 基于防火墙标记定义集群服务即可
ipvsadm -A -f 10  -s rr -p [600]

# PCC -- 端口定义为 0，此时必需使用 -p 选项
ipvsadm -A -t 192.168.1.99:0  -s rr -p [600]
```
