# 27.5 LVS DR模型实战
## 2. dr 类型演示示例
网络拓扑结构


### 2.1 lvs-dr 配置示例
```
# 1. 配置RS
# 1.1 配置 RS 拒绝对 VIP 的 ARP 广播地址请求 -- 修改两个内核参数
arp_announce: 是否接受其他主机的通告
    - =0: 表示有什么地址全部进行通告
    - =1: 尽量避免通过非本网络内的地址，但可以
    - =2: 只通告属于本网络内的地址
arp_ignore: 对ARP请求是否响应
    - =0: 对所有网卡回复所有地址的请求
    - =1: 仅回复对请求网卡上地址的请求
    - ....
# ---> 设置 arp_announce = 2, arp_ignore = 1
# 网卡接口参数配置在 /proc/sys/net/ipv4/conf/

echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce # 最好将所有接口都配置
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore

## 1.2 配置 vip 地址
ifconfig lo:0  192.168.1.121/32 broadcast 192.168.1.121 up
route add -host 192.168.1.121 dev lo:0 # 必须，以保证响应源 ip 为 vip

# 2. 配置 director
# 2.1 配置 vip
ifconfig ens33:0 192.168.1.121/32 broadcast 192.168.1.121 up
route add -host 192.168.1.121 dev ens33:0 # 非必须，保持和RS统一

# 3. 添加 ipvs 规则
ipvsadm -A -t 192.168.1.121 -s rr
ipvsadm -a -t 192.168.1.121:80 -r 172.16.1.102 -g
ipvsadm -a -t 192.168.1.121:80 -r 172.16.1.103 -g
```
