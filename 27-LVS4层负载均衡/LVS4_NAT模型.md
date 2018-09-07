# 27.4 LVS nat模型实战
本节我们就开始着手搭建一个 LVS-NAT 的负载均衡集群。

## 1. 网络拓扑结构


## 2. lvs-nat 配置
```
# 1. 配置 RS IP
vim /etc/sysconfig/network-scripts/ifcfg-ens33
setenforce 0
systemctl stop firewalld

# 2. 配置 RS httpd web server
# 3. 配置 Director ipvs
sysctl net.ipv4.ip_foward 1
ipvsadm -A -t 192.168.1.109:80 -s rr
ipvsadm -a -t 192.168.1.109:80 -r 172.16.1.102 -m
ipvsadm -a -t 192.168.1.109:80 -r 172.16.1.103 -m
ipvsadm -L -n

ipvadm -S -n    > /etc/sysconfig/ipvsadm
ipvsadm-save -n > /etc/sysconfig/ipvsadm

ipvsadm -C
ipvsadm -R < /etc/sysconfig/ipvsadm

# 修改规则
ipvsadm  -E -t 192.168.1.109:80 -s sh
ipvsadm -L -n

ipvsadm -e -t 192.168.1.109:80 -r 172.16.1.102:8080 -m # 不行
ipvsadm -d -t 192.168.1.109:80 -r 172.16.1.102
```
