# 24.7 nat路由高级实战进阶

## 1. 网络防火墙配置
### 1.1 打开网关Linux 主机的核心转发功能
- 内核: /proc/sys/net/ipv4/ip_forward
- 配置:
    - /etc/sysct.conf  --- net.ipv4.ip_forward = 1
    - systcl -w net.ipv4.ip_forward=1


## iptables-netfilter: nat table
如何让内网的主机访问互联网:
1. nat： 作用于网络层+传输层
2. proxy: 作用于应用层

## 1. nat 简介
### 1.1 nat 功能
- 定义: network address translation
    - snat: source nat 只修改请求报文的源地址
    - dnat: destination nat 只修改请求报文的目标地址
    - pnat: port nat
- snat：
    - POSTROUTING, OUTPUT
    - 让本地网络中的主机通过某一特定地址访问外部网络时；
- dnat：
    - PREROUTING
    - 把本地网络中的某一主机上的某服务开放给外部网络中的用户访问时；

### 1.2 nat表的target：
1. SNAT
    - --to-source [ipaddr[-ipaddr]][:port[-port]]
    - --random
2. DNAT
    - --to-destination [ipaddr[-ipaddr]][:port[-port]]
3. MASQUERADE
    - --to-ports port[-port]
    - --random
4. REDIRECT：端口重定向；
    - web: 8080 -- 80 --> 8080

```
# SNAT示例：
> iptables -t nat -A POSTROUTING -s 192.168.12.0/24 ! -d 192.168.12.0/24  -j SNAT --to-source 172.16.100.67    

# MASQUERADE示例：
# 源地址转换：当源地址为动态获取的地址时，MASQUERADE可自行判断要转换为的地址；
> iptables -t nat -A POSTROUTING -s 192.168.12.0/24 ! -d 192.168.12.0/24 -j MASQUERADE

# DNAT示例：
> iptables -t nat -A PREROUTING -d 172.16.100.67 -p tcp --dport 80 -j DNAT --to-destination 192.168.12.77

# REDIRECT
> iptables -t nat -A PREROUTING -d 172.16.100.67 -p tcp --dport 80 -j DNAT --to-destination 192.168.12.77:8080
> iptables -t nat -A PREROUTING -d 172.16.100.67 -p tcp --dport 22012 -j DNAT --to-destination 192.168.12.78:22
```
