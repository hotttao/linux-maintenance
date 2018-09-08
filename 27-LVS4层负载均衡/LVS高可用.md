# 27.7 LVS 高可用
前面我们搭建的负载均衡集群，一旦调度器发生故障，整个服务将不可用，我们需要对其进行高可用。keepalived 是 LVS 高可用最简单有效的解决方案，但是本节我们先不会讲解 keepalived，后面有一个章的内容专门讲解。本节的目的是通过对 LVS 高可用的讲解，让大家理解高可用集群中的重要概念，特别是健康状态检测。

## 1. LVS 的单点故障
LVS 的单点故障
1. Director不可用，整个系统将不可用，这是整个集群的单点故障所在
  - 解决方案: 高可用
  - 实现: keepalived，heartbeat/corosync
2. 某RS不可用时，Director依然会调度请求至此RS；
  - 解决方案：对各RS的健康状态做检查，失败时禁用，成功时启用；
  - 实现: keepalived, heartbeat/corosync, ldirectord

对于后端服务器的健康状态检测应该是调度器本身所具有的功能，但是因为 LVS 工作的太多底层，所以 LVS 本身不具有此功能，其需要借助外部工具来实现。

keepalived 是帮助  LVS 实现高可用的，但是额外也能帮助 LVS 实现健康状态检测，并且在服务器状态发生变化时，完成 ipvs 对服务器的增删操作。ldirectord 则主要就是为了帮助 LVS 做后端状态检测，并且在服务器状态发生变化时，完成对服务器的增删操作，除此之外没有别的功能。

### 1.1 健康状态检测
对服务器的健康状态检测理解，可以从检查机制，检查后的操作进行理解
#### 检查机制
检查机制: 又称检查方法，即通过什么方式，怎么判断服务器已经故障或已恢复
1. 检查方法:
  - 网络层检测: ping 主机
  - 传输层检测: 端口探测，检测服务端口是否存在
  - 应用层检测: 某关键资源是否能被请求到
2. 判断方式: 很显然，我们不能因为某一次检测失败就判定后端服务器故障，因为有可能网络出现问题，也有可能服务器繁忙还没来得及响应。因此我们需要经过多次检测结果来判断服务器的状态。

```
# 服务器状态转换
第一次故障时 --------> 软状态  ----> 多次故障 ------> 硬状态(真正认定为故障)
```

## 2. ldirectord
### 2.1 安装
ldirectord 的rpm 包下载链接
ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/network:/ha-clustering:/Stable/CentOS_CentOS-6/x86_64/ldirectord-3.9.5-3.1.x86_64.rpm

```
$ rpm -ql ldirectord
/etc/ha.d                          # 配置文件目录
/etc/ha.d/resource.d
/etc/ha.d/resource.d/ldirectord   # 主程序链接
/etc/init.d/ldirectord
/etc/logrotate.d/ldirectord
/usr/lib/ocf/resource.d/heartbeat/ldirectord
/usr/sbin/ldirectord                    # 主程序
/usr/share/doc/ldirectord-3.9.5
/usr/share/doc/ldirectord-3.9.5/COPYING
/usr/share/doc/ldirectord-3.9.5/ldirectord.cf # 配置文件示例
```

### 2.2 配置
需要在注意的是  ldirectord 会自动根据配置文件及后端服务器可用状态生成规则，因此无需在使用 ipvsadm 创建集群。

```
cp /usr/share/doc/ldirectord-3.9.5/ldirectord.cf  /etc/ha.d/

vim  /etc/ha.d/ldirectord.cf

# 配置示例
# Global Directives
checktimeout=3           # 检测的超时时长
checkinterval=1          # 检测的频率，单位秒
fallback=127.0.0.1:80    # 所有后端服务器都不可用，最后的备用服务器，sorry server
autoreload=yes           # 配置文件修改时，是否自动加载
logfile="/var/log/ldirectord.log"
#logfile="local0"
#emailalert="admin@x.y.z" # 通知管理员
#emailalertfreq=3600    # 故障未修复，每隔多长时间发一次邮件
#emailalertstatus=all # 对哪些状态改变发送邮件
quiescent=no

# virtual 对应于 LVS 一个集群
virtual=3
  real=192.168.1.107:80 gate 2  # RS
  real=192.168.1.109:80 gate 1
  fallback=127.0.0.1:80 gate
  service=http
  scheduler=wrr
  #persistent=600
	#netmask=255.255.255.255

  checktype=negotiate
  checkport=80
  request="index.html"
  #receive="CentOS"
  #virtualhost=www.x.y.z # 向哪个 httpd 的虚拟机主机发送请求
```
