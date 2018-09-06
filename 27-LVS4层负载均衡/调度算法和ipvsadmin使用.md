# 27.3 LVS 调度算法和 ipvsadmin 命令使用
上一节我们对 LVS 的工作原理和四种模型下如何实现负载均衡做了简单介绍，本节我们来学习 LVS 种可用的调度算法以及 ipvsadmin 命令的使用。
## 1. lvs 调度算法
LVS 的调度算法分为静态方法和动态方法两类
### 1.1 静态算法
静态方法: 仅根据算法本身进行调度
1. `RR`: round robin, 轮调
2. `WRR`: weighted rr, 加权轮调
3. `SH`: source hash, 源地址哈希，实现 session 保持的机制 -- 来自同一个IP的请求始终调度至同一RS
4. `DH`: destination hash，目标地址哈希，将对同一个目标的请求始终发往同一RS

### 1.2 动态算法
动态方法: 根据算法及各 RS 的当前负载(Overhead)状态进行调度
1. `LC`: Least Connection
    - Overhead = Active * 256 + Inactive
2. `WLC`: Weighted LC
    - Overhead = (Active * 256 + Inactive) / weight
3. `SED`: Shortest Expection Delay
    - Overhead = (Active + 1) * 256 / weight
4. `NQ`: Never Queue, 按照 SED 进行调度，但是被调度的主机，在下次调度时不会被选中 -- SED 算法改进
5. `LBLC`:
    - 定义: Locality-Based LC，即动态的 DH 算法
    - 作用: 正向代理情形下的 cache server 调度
6. `LBLCR`:
    - 定义: Locality-Based Least-Connection with Replication 带复制的LBLC算法
    - 特性: 相对于 LBLC，缓存服务器之间可以通过缓存共享协议同步缓存


## 2. ipvsadm
### 2.1 ipvsadmin 简介
使用 ipvsadmin 定义一个负载均衡集群时
1. 首先要定义一个集群，然后向集群内添加 RS。
2. 一个 ipvs 主机可以同时定义多个集群服务
3. 一个 cluster server 上至少应该有一个 real server

在适用 ipvsadmin 定义集群服务之前，首先要确定 ipvs 已在内核中启用。Centos 的 `/boot/config-VERSION` 文件内记录了编译内核的所有参数，通过此文件查看 ipvs 配置参数即可确定 ipvs 是否启用。

```bash
# 查看 ipvs 在内核中是否启用，及其配置
$ grep -i -A 10 "IP_VS" /boot/config-3.10.0-514.el7.x86_64

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

### 2.2 ipvsadmin 程序包组成
```
$ yum install ipvsadm
$ rpm -ql ipvsadm
/etc/sysconfig/ipvsadm-config              # 默认的规则保存文件
/usr/lib/systemd/system/ipvsadm.service    # unit file
/usr/sbin/ipvsadm                          # 主程序
/usr/sbin/ipvsadm-restore                  # 规则保存工具
/usr/sbin/ipvsadm-save                     # 规则重载工具
```

ipvs 直接附加在内核之上，只要内核正常运行，ipvs 即可工作。ipvs 的 Unit file 主要是在启动时加载规则，在关闭时保存规则而已

```
# cat /usr/lib/systemd/system/ipvsadm.service
[Unit]
Description=Initialise the Linux Virtual Server
After=syslog.target network.target

[Service]
Type=oneshot
# start 加载规则
ExecStart=/bin/bash -c "exec /sbin/ipvsadm-restore < /etc/sysconfig/ipvsadm"

# stop 保存规则
ExecStop=/bin/bash -c "exec /sbin/ipvsadm-save -n > /etc/sysconfig/ipvsadm"
ExecStop=/sbin/ipvsadm -C
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```


### 2.3 ipvsadm 使用
ipvsadm命令的核心功能：
1. 集群服务管理：增、删、改；
2. 集群服务的RS管理：增、删、改；
3. 集群的查看

#### 集群服务管理
`ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]]`
- 作用: 集群服务的增，改
- 选项:
  - `-A`: 添加集群服务
  - `-E`: 修改集群服务
  - `-t|u|f service-address`: 指定集群的作用的协议，地址和端口，唯一标识一个集群
    - `-t`: TCP协议 VIP:TCP_PORT
    - `-u`: UDP协议，VIP:UDP_PORT
    - `-f`：firewall MARK，是一个数字
  - `-s scheduler`: 调度算法，默认为 `wlc`

`ipvsadm -D -t|u|f service-address`
- 作用: 删除集群服务
- 选项:
  - `-t|u|f service-address`: 指定删除的集群

`ipvsadm -C`
- 作用: 清空定义的所有内容

#### 管理集群服务上的 RS
`ipvsadm -a|e -t|u|f service-address -r server-address [-g|i|m] [-w weight]`
- 作用: 添加或修改集群服务的 RS
- 选项:
  - `-a`: 添加 RS
  - `-e`: 修改 RS
  - `-t|u|f service-address`:指定管理的集群
  - `-r server-address[:port]`: 指定 RS 的 ip 地址端口
  - `-g|i|m`: 指定lvs类型，默认为 m
    - `-g`: gateway, dr类型
    - `-i`: ipip, tun类型
    - `-m`: masquerade, nat类型
  - `-w weight`: 权重


`ipvsadm -d -t|u|f service-address -r server-address`
- 作用: 删除集群服务上的 RS
- 选项:
  - `-t|u|f service-address`:指定管理的集群
  - `-r server-address[:port]`: 指定 RS 的 ip 地址端口

#### 查看
`ipvsadm -L|l [options]`
- 作用: 查看集群状态信息
- 选项:
  - `--numeric, -n`: 基于数字格式显示ip和端口
  - `--connection，-c`: 显示当前的连接
  - `--exact`: 显示统计数据精确值  
  - `--stats`: 显示统计数据
  - `--rate` : 显示速率

`ipvsadm -Z [-t|u|f service-address]`
- 作用: 清空集群的计数器
- 选项:
  - `-t|u|f service-address`:指定管理的集群

#### 规则保存和重载
`ipvsadm -R`
- 作用: 重载 == `ipvsadm-restore`

`ipvsadm -S [-n]`
- 作用: 保存 == `ipvsadm-save`
