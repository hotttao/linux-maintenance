# 29.2 keepalived 安装和配置
上一节我们对高可用集群 和 keepalived 做了一个简单介绍，本节我们来学习 keepalived 的安装配置。我们的最终目的是使用 keepalived 对 nginx 的负载均衡集群做高可用，本节内容包括:
1. HA 集群配置的前题
2. keepalived 安装与组成
3. keepalived 配置文件格式与参数

## 1. HA 集群的配置前提
HA 集群因为主备节点之间需要通信以协调工作，所以在配置之前需要一些准备工作:
1. 各节点时间必须同步，参考 [25.1 Linux时间服务-chrony](../25-Linux日志-时钟服务-sudo/Linux时间服务-chrony.md)
2. 确保iptables及selinux不会成为阻碍
3. 各节点之间可通过主机名互相通信
  - 对 keepalived 并非必须，但是对于 heartbeat/corosync 则是必备条件
  - 建议使用/etc/hosts文件实现，避免 DNS 称为单点故障所在
4. 确保各节点的用于集群服务的网卡接口支持MULTICAST通信，以便进行组播
5. 各节点之间的 root 用户可以基于密钥认证的 ssh 服务完成互相通信。
  - 对 keepalived 并非必须，但是对于 heartbeat/corosync 则是必备条件
  - 因为 corosync 需要在节点之间复制配置文件

## 2. keepalived
CentOS 6.4 只有 keepalived 就已经被收录至 base 仓库，因此可通过 yum 直接安装。

## 2.1 程序环境
```bash
$ rpm -ql keepalived |grep -v "share"
/usr/sbin/keepalived              # 主程序文件
/etc/keepalived
/etc/keepalived/keepalived.conf   # 主配置文件
/usr/bin/genhash                  
/etc/sysconfig/keepalived         # Unit File的环境配置文件
/usr/lib/systemd/system/keepalived.service # Unit File
/usr/libexec/keepalived
```

## 2.2 配置文件格式
```bash
# 1. GLOBAL CONFIGURATION
# 1.1 Global definitions
global_defs {
}

# 1.2 Static routes/addresses
static_ipaddress{
}

static_routes{

}

static_rules{

}

# 2. VRRPD CONFIGURATION
# 2.1 VRRP synchronization group(s)
vrrp_sync_group VG_1 {
  group {
    inside_network   # name of the vrrp_instance (see below)
    outside_network  # One for each movable IP
 ...}
}

# 2.2 VRRP instance(s)
vrrp_instance inside_network {

}

# 3. LVS CONFIGURATION
# 3.1 Virtual server group(s)
virtual_server_group <STRING> {

}

# 3.2 Virtual server(s)
virtual_server group string{

}
```
上面是 `keepalived.conf` 的缩略结构，主要由如下几个配置段组成:
1. `GLOBAL CONFIGURATION`: 全局配置段
  - `Global definitions`: 全局参数
  - `Static routes/addresses`: 静态地址和静态路由配置
2. `VRRPD CONFIGURATION`: vrrp 配置段
  - `VRRP synchronization group(s)`：vrrp同步组，同一组内的 vrrp 会同进同退
  - `VRRP instance(s)`：每个vrrp instance即一个vrrp路由器；
3. `LVS CONFIGURATION`: lvs 规则管理配置段
  - `Virtual server group(s)`: 将一组集群服务进行统一调度
  - `Virtual server(s)`: ipvs集群的vs和r

## 3. global_defs
global_defs 用于全局参数

```
# 示例一: global_defs 全局配置
! Configuration File for keepalived

global_defs {
  notification_email {   # 邮件通知的管理员帐户，收件箱
    root@localhost
  }
  notification_email_from keepalived@localhost # 发件箱
  smtp_server 127.0.0.1                        
  smtp_connect_timeout 30                      # smtp 链接超时时长
  router_id node1                              # 当前节点的标识，重要
  vrrp_mcast_group4 224.0.100.19               # 组播域，重要
}
```

## 4. vrrp_instance
`vrrp_instance` 用于定义虚拟路由器

```
vrrp_instance <STRING> {
  ....
}
```
### 4.1 vrrp_instance 常用参数
|vrrp_instance 参数|作用|
|:---|:---|
|`state MASTER/BACKUP`|当前节点在此虚拟路由器上的初始状态；只能有一个是MASTER，余下的都应该为BACKUP|
|`interface IFACE_NAME`|绑定为当前虚拟路由器使用的物理接口|
|`virtual_router_id VRID`|当前虚拟路由器的惟一标识，范围是0-255|
|`priority 100`|当前主机在此虚拟路径器中的优先级；范围1-254|
|`advert_int 1`|vrrp通告的时间间隔；|
|`authentication{}`|vrrp 认证，详细使用见下|
|`virtual_ipaddress{}`|虚拟路由器的 IP 地址，详细使用见下|
|`virtual_routes{}`|虚拟路由，详细使用见下|
|`track_interface{}`|配置要监控的网络接口，一旦接口出现故障，则转为FAULT状态,，详细使用见下|
|`nopreempt`|定义工作模式为非抢占模式，默认为抢占模式|
|`preempt_delay 300`|抢占式模式下，节点上线后触发新选举操作的延迟时长；|
|`notify_master path`|当前节点成为主节点时触发的脚本|
|`notify_backup path`|当前节点转为备节点时触发的脚本|
|`notify_fault path`|当前节点转为“失败”状态时触发的脚本|
|`notify path`|通用格式的通知触发机制，一个脚本可完成以上三种状态的转换时的通知|

```
# vrrp 认证
authentication {
      auth_type AH|PASS
      auth_pass <PASSWORD>
    }

# 虚拟路由器 ip
virtual_ipaddress {
  <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
  192.168.200.17/24 dev eth1
  192.168.200.18/24 dev eth2 label eth2:1
}

# 虚拟路由
virtual_routes {
     src 192.168.100.1 to 192.168.109.0/24 via  192.168.200.254  dev eth1
     192.168.110.0/24 via 192.168.200.254 dev eth1
 }


# 监控的网络接口
track_interface {
  eth0
  eth1
  ...
}
```
### 4.2 vrrp_instance 配置示例
```bash
# 示例二: 单主模型下完成地址流动
global_defs{          # 配置见示例一
  .....
}

vrrp_instance VI_1 {
  state BACKUP             # 节点初始状态
  interface eno16777736    # 绑定虚拟地址的网卡接口
  virtual_router_id 14     # 虚拟路由器 ID
  priority 98              # 优先级
  advert_int 1             # 组播频率
  authentication {         # vrrrp 认证
    auth_type PASS
    auth_pass 571f97b2
  }
  virtual_ipaddress {      # 虚拟 IP 地址
    10.1.0.91/16 dev eno16777736 label eno16777736:0
  }
}			
```

## 5. vrrp_sync_group
### 5.1 vrrp_sync_group 作用
`VRRP synchronization group(s)`：用于定义 vrrp 同步组，同一组内的 vrrp 会同进同退。所谓同进同退的意思是 vrrp 组的服务，当某一个服务发生故障转移或故障恢复时，组内的所有服务都会一同进行转移。典型的情景是高可用 NAT 模型的 LVS。

```
  vip ----------- VS1(100) ------ DIP
  vip ------------VS2(99) -------DIP
```

如上，前段我们将 vip 定义为虚拟路由 router1，对外提供服务，后端我们将 DIP 配置为虚拟路由器 router2 向后端服务转发请求。当 router1 因为某种原因从 VS1 转移到 VS2 时，我们的 router2 也必需要转移过去。原因是 NAT 模型的 LVS 后端的 RS 必需将网关指向VS，当 VS 由 VS1 转移到 VS2 时，如果 router2 不随之转移，RS 的报文将将默认发送至 VS1，此时将无法完成目标地址转换。

需要注意的是对于 nginx 我们无需配置 router2，因为请求报文是通过 IP 地址路由的，而 IP 地址是不会变化的。

### 5.2 vrrp_sync_group 配置示例
```
vrrp_sync_group G1 {
     group {          
       VI_1    # vrrp_instance 定义 vrrp 虚拟路由器名称
       VI_2
       VI_5
}
```

## 6. virtual_server
virtual_server 用于定义 ipvs 集群规则
```
virtual_server IP port |         # 只支持 tcp 协议
virtual_server fwmark int        # 防火墙标记
{
  ...
  real_server {
    ...
  }
  ...
}
```

### 6.1 virtual_server 常用参数
1. `delay_loop <INT>`：健康状态检测的时间间隔
2. `lb_algo rr|wrr|lc|wlc|lblc|sh|dh`：调度方法
3. `lb_kind NAT|DR|TUN`：集群的类型
4. `persistence_timeout <INT>`：持久连接时长
5. `protocol TCP`：服务协议，仅支持TCP
6. `sorry_server <IPADDR> <PORT>`：备用服务器地址
7. `real_server <IPADDR> <PORT>{}`：RS 定义


#### RS 定义
```
real_server <IPADDR> <PORT>
{
 weight <INT>                         : 权重
 notify_up <STRING>|<QUOTED-STRING>   : 启动的通知脚本
 notify_down <STRING>|<QUOTED-STRING> : 关闭的通知脚本
 HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK { ... }：定义当前主机的健康状态检测方法；
}
```

健康状态检测方法:
1. `HTTP_GET|SSL_GET{}`：应用层检测
2. `TCP_CHECK{}`：传输层检测

#### 应用层检测
位于 `real_server{}`配置段内

```
HTTP_GET|SSL_GET {
    url {
      path <URL_PATH>        ：定义要监控的URL；
      status_code <INT>      ：判断上述检测机制为健康状态的响应码；
      digest <STRING>        ：判断上述检测机制为健康状态的响应的内容的校验码；
    }
    nb_get_retry <INT>       ：重试次数；
    delay_before_retry <INT> ：重试之前的延迟时长；
    connect_ip <IP ADDRESS>  ：向当前RS的哪个IP地址发起健康状态检测请求
    connect_port <PORT>      ：向当前RS的哪个PORT发起健康状态检测请求
    bindto <IP ADDRESS>      ：发出健康状态检测请求时使用的源地址；
    bind_port <PORT>         ：发出健康状态检测请求时使用的源端口；
    connect_timeout <INTEGER>：连接请求的超时时长；
}
```

#### 传输层检测
位于 `real_server{}`配置段内

```
TCP_CHECK {
    connect_ip <IP ADDRESS>   ：向当前RS的哪个IP地址发起健康状态检测请求
    connect_port <PORT>       ：向当前RS的哪个PORT发起健康状态检测请求
    bindto <IP ADDRESS>       ：发出健康状态检测请求时使用的源地址；
    bind_port <PORT>          ：发出健康状态检测请求时使用的源端口；
    nb_get_retry <INT>        ：重试次数；
    delay_before_retry <INT>  ：重试之前的延迟时长；
    connect_timeout <INTEGER> ：连接请求的超时时长；
}
```

### 6.2 配置示例
```
virtual_server 10.1.0.93 80 {
  delay_loop 3                     # 健康状态监测时间间隔
  lb_algo rr                       # 调度算法
  lb_kind DR                       # 集群类型
  protocol TCP                     # 服务协议

  sorry_server 127.0.0.1 80        # sorry server

  real_server 10.1.0.69 80 {       # RS 配置
    weight 1                       # 权重
    HTTP_GET {                     # 应用层健康状态监测
    url {
      path /                       # 检测路经
      status_code 200
    }
    connect_timeout 1              # 链接超时时长
    nb_get_retry 3                 # 重试次数
    delay_before_retry 1           # 重试间隔
    }
  }
  real_server 10.1.0.71 80 {
    weight 1
    HTTP_GET {
    url {
      path /
      status_code 200
    }
    connect_timeout 1
    nb_get_retry 3
    delay_before_retry 1
    }
  }
}
```

## 7. keepalived 高可用 nginx
keepalived 最初的设计目的是为了高可用 LVS，所以要想高可用其他服务需要借助于 keepalived 的脚本调用接口。keepalived 通过调用外部的辅助脚本进行资源监控，并根据监控的结果实现节点的优先调整，以便在主节点发生故障时实现故障转移。

对于 nginx 调度器为例，其最重要的资源是对外提供服务的 IP 地址和 nginx 进程，keepalived 的 vrrp stack 已经能自动完成 IP 转移，但是 keepalived 并没有内置判断 nginx 是否故障，以及故障之后如何转移的功能。nginx 资源的监控，以及如何进行优先级调整只能通过提供辅助脚本进行。并且此时后端服务器的健康状态检测由 nginx 自己进行，与 keepalived 无关。

因此使用 keepalived 高可用 nginx 分两步：
1. 先定义一个 nginx 的监控脚本，使用 keepalived 的 `vrrp_script{}` 配置段
2. 调用此脚本，在 `vrrp_instance{}` 配置段内使用 `track_script{}` 配置段

### 7.1 vrrp_script
```
vrrp_script <SCRIPT_NAME> {
  script ""        # 脚本路经
  interval INT     # 脚本执行的时间间隔
  weight -INT      # 脚本执行失败后，对优先级的调整大小
  fall INT         # 认定失败的检测次数
  rise INT         # 认定恢复正常的检测次数
  user USERNAME [GROUPNAME]  # 执行脚本的用户和用户组
}
```

### 7.2 track_script
```
track_script {
  SCRIPT_NAME_1  # vrrp_script 定义的脚本名称
  SCRIPT_NAME_2
  ...
}
```

### 7.3 配置示例
```
vrrp_script chk_down {
  script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
  interval 1
  weight -5
}

vrrp_script chk_nginx {
  script "killall -0 nginx && exit 0 || exit 1"
  interval 1
  weight -5
  fall 2
  rise 1
}

vrrp_instance VI_1 {
  state MASTER
  interface eno16777736
  virtual_router_id 14
  priority 100
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 571f97b2
  }
  virtual_ipaddress {
    10.1.0.93/16 dev eno16777736
  }
  track_script {
    chk_down
    chk_nginx
  }
  notify_master "/etc/keepalived/notify.sh master"
  notify_backup "/etc/keepalived/notify.sh backup"
  notify_fault "/etc/keepalived/notify.sh fault"
}
```
