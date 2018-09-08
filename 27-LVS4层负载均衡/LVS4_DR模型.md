# 27.5 LVS DR模型实战
本节我们将搭建一个 LVS-DR 的负载均衡集群。
## 1. 网络拓扑结构
![web_fram](../images/27/lvs-dr-frame.png)

拓扑结构说明:
- VS， RS1， RS2 在虚拟机内均采用桥接方式，桥接到物理机的网卡上
- VIP 配置在 DIP 所在网卡的别名上

## 2. lvs-dr 配置示例
### 2.1 RS 配置脚本
```bash
#!/bin/bash

vip="192.168.1.99"

case $1 in
start)
	echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
	echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
	echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
	echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
  iptables -F
  ifconfig lo:0 $vip netmask 255.255.255.255 broadcast $vip up
	route add -host $vip dev lo:0
	;;
stop)
	ifconfig lo:0 down
	echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
	echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
	echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
	echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
	;;
*)
  echo "Usage $(basename $0) start|stop"
  exit 1
  ;;
esac

echo "/proc/sys/net/ipv4/conf/all/arp_ignore:"   `cat  /proc/sys/net/ipv4/conf/all/arp_ignore`
echo "/proc/sys/net/ipv4/conf/lo/arp_ignore:"   `cat  /proc/sys/net/ipv4/conf/lo/arp_ignore`
echo "/proc/sys/net/ipv4/conf/all/arp_announce:"   `cat  /proc/sys/net/ipv4/conf/all/arp_announce`
echo "/proc/sys/net/ipv4/conf/lo/arp_announce:"   `cat  /proc/sys/net/ipv4/conf/all/arp_announce`
```

### 2.2 VS 配置脚本
```bash
#!/bin/bash
vip="192.168.1.99"
ifc="enp0s3:0"
port=80
rs1="192.168.1.107"
rs2="192.168.1.109"

case $1 in
start)
	ifconfig $ifc $vip netmask 255.255.255.255 broadcast $vip up
	iptables -F
	ipvsadm -A -t $vip:$port -s wrr
	ipvsadm -a -t $vip:$port -r $rs1 -g -w 1 	
	ipvsadm -a -t $vip:$port -r $rs2 -g -w 1
	;;
stop)
	ipvsadm -C
	ifconfig $ifc down
	;;
*)
  echo "Usage $(basename $0) start|stop"
  exit 1
  ;;
esac
```
