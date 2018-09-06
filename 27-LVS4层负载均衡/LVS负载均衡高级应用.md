# 27.6 LVS 高级应用配置实战
## 1. 虚拟网络接口类型
1. bridge: 将当前主机的物理网卡与VMware 内部的虚拟交换机进行关联
2. nat: 在当前物理主机的物理网卡上启动 nat 转发功能，转发内部虚拟主机的网络请求，以连接外部物理主机
3. host-only:
    - 功能: 虚拟主机能与其他虚拟主机以及当前物理主机进行通信，不能与外部物理主机通信
    - 特征: 与 nat 相比，仅仅是取消了当前物理网卡的 nat 功能
4. 私有网桥(VMnet2)
    - 功能: 仅虚拟主机之间可以通信，不能与当前物理主机通信


## 3. FWM -- 防火墙标记
- 作用: 将共享一组RS的集群服务统一进行定义
- 配置:
    1. 在 director 上 netfilter 的 mangle 表的 PREROUTING 链上，定义防火墙标记规则
    2. 基于FWM 定义集群服务

```
# 对服务进行防火墙标记
iptables -t mangle -A PREROUTING -d 192.168.0.10 -p tcp --dport 80 -j MARK --set-mark 10

# 通过防火墙标记，定义集群服务
ipvsadm -A -f 10 -s rr  
ipvsadm -a -f 10 -r 172.168.100.21 -g
ipvsadm -a -f 10 -r 172.168.100.22 -g

ipvsadm -L -n
# 扩展集群服务
iptables -t mangle -A PREROUTING -d 192.168.0.10 -p tcp --dport 22 -j MARK --set-mark 10
```

## 4. lvs persistence -- lvs 持久连接
- 功能: 无论 ipvs 使用何种调度算法，其都能实现将来自同一Client的请求始终定向至第一次调度时挑选的RS
- 说明:
    - lvs sh 算法: 仅能在某一特定服务内，实现 session 绑定
    - lvs persistenc: 对多个共享同一组RS的服务器，实现统一 session 绑定
- 实现: 持久连接模板，独立于调度算法  `sourceip  rs  timer`
- 类型:
    - PPC: 每端口持久，单服务持久调度
    - PFWMC: 每FWM持久, 单 FWM 持久调度
    - PCC: 每客户端持久，单客户端持久调度；director 会将用户的**任何请求都识别为集群服务**，并向RS进行调度

```
# PPC: -p 表示持久连接，后跟持久连接时长，默认 360s
ipvsadm -A -t 192.168.0.10:80  -s rr -p [600]
ipvsadm -a -t 192.168.0.10:80  -r 172.168.100.21 -g
ipvsadm -a -t 192.168.0.10:80  -r 172.168.100.22 -g

# PCC -- 端口定义为 0
ipvsadm -A -t 192.168.0.10:0  -s rr -p [600]

# PFWMC -- 基于防火墙标记定义集群服务即可
ipvsadm -A -f 10  -s rr -p [600]
```
