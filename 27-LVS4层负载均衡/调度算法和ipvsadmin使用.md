# 27.3 LVS 调度算法和 ipvsadmin 命令使用

## 1. lvs 调度算法
### 1.1 session 保持方法
1. session 绑定:
    - 定义: 将来自同一用户的请求始终调度到同一个 RS
    - 方法: source ip_hash/cookie hash
    - 问题: 主机迭机之后，该主机上的所有 session 也会丢失
2. session 集群:
    - 定义: 每一台主机上都保留了所有用户的 session
    - 问题: session 同步问题
3. session 服务器:
    - 定义: 第三方服务持久化存储 session

### 1.2 lvs scheduler
静态方法: 仅根据算法本身进行调度
1. RR: round robin, 轮调
2. WRR: weighted rr, 加权轮调
3. SH: source hash, 源地址哈希，实现 session 保持的机制 -- 来自同一个IP的请求始终调度至同一RS
4. DH: destination hash，目标地址哈希，将对同一个目标的请求始终发往同一RS

动态方法: 根据算法及各 RS 的当前负载(Overhead)状态进行调度
1. LC: Least Connection
    - Overhead = Active * 256 + Inactive
2. WLC: Weighted LC
    - Overhead = (Active * 256 + Inactive) / weight
3. SED: Shortest Expection Delay
    - Overhead = (Active + 1) * 256 / weight
4. NQ: Never Queue, 按照 SED 进行调度，但是被调度的主机，在下次调度时不会被选中 -- SED 算法改进
5. LBLC:
    - 定义: Locality-Based LC，即动态的 DH 算法
    - 作用: 正向代理情形下的 cache server 调度
6. LBLCR: Locality-Based Least-Connection with Replication 带复制的LBLC算法


## 2. ipvsadm 的用法
两步骤:
1. 管理集群服务
2. 管理集群服务中的RS

ipvs 集群服务
- 一个 ipvs 主机可以同时定义多个 cluster service
- 一个 cluster server 上至少应该有一个 real server
- 定义时，要指明 lvs-type，以及 lvs scheduler

### 2.1 ipvs 是否启动
```
# 查看 ipvs 在内核中是否启用，及其配置
grep -i -A 10 "IP_VS" /boot/config-3.10.0-514.el7.x86_64

CONFIG_IP_VS=m
CONFIG_IP_VS_IPV6=y
# CONFIG_IP_VS_DEBUG is not set
CONFIG_IP_VS_TAB_BITS=12

#
# IPVS transport protocol load balancing support
#
CONFIG_IP_VS_PROTO_TCP=y
CONFIG_IP_VS_PROTO_UDP=y
CONFIG_IP_VS_PROTO_AH_ESP=y
CONFIG_IP_VS_PROTO_ESP=y
CONFIG_IP_VS_PROTO_AH=y
CONFIG_IP_VS_PROTO_SCTP=y

#
# IPVS scheduler
#
CONFIG_IP_VS_RR=m
CONFIG_IP_VS_WRR=m
CONFIG_IP_VS_LC=m
CONFIG_IP_VS_WLC=m
CONFIG_IP_VS_LBLC=m
CONFIG_IP_VS_LBLCR=m
CONFIG_IP_VS_DH=m
CONFIG_IP_VS_SH=m
CONFIG_IP_VS_SED=m
CONFIG_IP_VS_NQ=m
```




### 9.1 管理集群服务
```
# 集群服务增，改，删，查
ipvsadm -A|E -t|u|f service-address [-s scheduler]
              [-p [timeout]] [-M netmask] [-b sched-flags]
ipvsadm -D -t|u|f service-address
ipvsadm -C
ipvsadm -L|l [options]
        -n: 基于数字格式显示ip和端口
        -c: 显示当前已经建立的TCP连接
        --state: 显示统计数据
        --rate:  显示速率
        --exact: 显示统计数据精确值
```

service-address
- tcp: -t ip:port
- udp: -u ip:port
- fwm: -f mark

### 9.2 管理集群服务中的 RS  
```
# 集群服务中的RS 增，改，删
ipvsadm -a|e -t|u|f service-address -r server-address
      [-g|i|m] [-w weight] [-x upper] [-y lower]
ipvsadm -d -t|u|f service-address -r server-address

# 清空和查看
ipvsadm -C             
ipvsadm -L|l [options]

ipvsadm -R      # 重载 == ipvsadm-restore
ipvsadm -S [-n]  # 保存 == ipvsadm-save
ipvsadm -Z [-t|u|f service-address]  # 清空计数器
```
参数:
- server-address: ip[:port]
- lvs-type:
    - -g: gateway, lvs-dr 模型(默认)
    - -i: ipip, lvs-tun 模型
    - -m: masquerade, lvs-nat 模型
- -s scheduler: 指定调度算法，默认 WLC
