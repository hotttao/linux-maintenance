# 12.3 网络属性配置之 ifcfg 系列命令
本节我们来介绍 Linux 网络属性配置的第一组系列命令 ifcfg。ifcfg 系列是 Linux 中很古老的命令，几乎存在于所有的Linux 发行版中。ifcfg 系列包括如下几个命令:
- `ifconfig`：配置IP，NETMASK
- `route`：路由查看与管理
- `netstat`：网络状态及统计数据查看
- `ifup/ifdown`: 启动/关闭接口

## 1. ifconfig
ifconfig 的配置会立即送往内核中的TCP/IP协议栈，并生效

#### 查看接口状态
- `ifconfig`：默认显示活跃状态的接口
- `ifconfig IFACE [up|down]`：后跟网络接口名，显示特定接口，up 表示激活接口，down 表示关闭接口
- `ifconfig -a`：显示所有接口，包括inactive状态的接口；

#### 设置接口地址
`ifconfig IFACE {address [netmask NETMASK]} options`
- `ifconfig  IFACE  IP/MASK`
- `ifconfig  IFACE  IP netmask  NETMASK `  
- options：
    - `up`： 激活接口。如果给接口声明了地址，等于隐含声明了这个选项
    - `down`: 关闭此接口
    - `[-]promisc`: 启用混杂模式，减号表示禁用
    - `add addr/prefixlen`：添加 IPV6 地址
    - `del addr/prefixlen`：删除 IPV6 地址

```
ifconfig eth0 192.168.100.6/24  up
ifconfig eth0 192.168.100.6 netmask 255.255.255.0
```

## 2 route
路由表中的路由条目有三种类型，范围越小优先级越高:
- 主机路由：目标地址为单个IP；
- 网络路由：目标地址为IP网络；
- 默认路由：目标为任意网络，0.0.0.0/0.0.0.0

#### 路由查看
`route  -n`
- `-n`: 以数字形式显示主机名，默认会反解主机名

#### 添加路由
`route  add  [-net|-host]  target  [netmask  Nm]  [gw GW]  [[dev] If]`
- `-net`: 指定网络路由
- `-host`: 指定主机路由
- `netmask`: 指定掩码, 默认为 255.255.255.255
- `gw`: 指定路由的下一跳
- `dev`: 指定发送数据包的网卡

```
示例：
route add -host 192.168.100.6  gw 192.168.0.1 dev eth0
route add -net  10.0.0.0/8  gw  192.168.10.1  dev  eth1
route add -net  10.0.0.0 netmask 255.0.0.0  gw  192.168.10.1  dev  eth1
route add -net  0.0.0.0/0.0.0.0  gw 192.168.10.1  
route add default  gw 192.168.10.1  -- 添加默认路由
route add -net 0.0.0.0  netmask 0.0.0.0 gw 192.168.10.1 -- 添加默认路由
```            

#### 删除路由
`route  del  [-net|-host] target  [gw Gw]  [netmask Nm]  [[dev] If]`

```
示例：
route del -host 192.168.100.6  gw 192.168.0.1
route del -net  10.0.0.0/8  gw 192.168.10.1
route del default
```                          

## 3. netstat
netstat can print network connections, routing tables, interface statistics, masquerade connections, and multicast  memberships

#### 显示路由表
`netstat  -rn`
- `-r`：显示内核路由表
- `-n`：以数字形式显示主机名，默认会反解主机名

#### 显示网络连接
`netstat [options]`
- 选项:
    - `-t, --tcp`：显示TCP协议的相关连接，及其链接状态；
    - `-u, --udp`：显示UDP相关的连接
    - `-U, --udplite`：显示udplite 相关的链接
    - `-S, --sctp`：显示 sctp 相关的链接
    - `-w, --raw`：显示raw socket相关的连接,指不经过传输层，由应用层直接通过 ip 进行的链接
    - `-l, --listening`：显示处于监听状态的连接
    - `-a, --all`：显示所有状态
    - `-n, --numeric`：以数字格式显示IP和Port；
    - `-e, --extend`：以扩展格式显示  
    - `-p, --program`：显示相关的进程及PID；

#### 显示接口的统计数据
`netstat {--interfaces|-I|-i} [options]`
- 选项:
    - `-I, --interfaces<iface>`: 指定显示的接口 `eg: netstat -Ietho`
    - `-i`: 显示所有活跃接口
    - `-a, --all`: 与 `-i` 同时使用显示所有接口，包括未激活的
    - `-e, --extend`: 以扩展格式进行显示

### 4. ifup/ifdown
`ifup|ifdown iface`
- 作用: 启用或关闭接口
- 注意: 这两个命令是通过配置文件`/etc/sysconfig/network-scripts/ifcfg-IFACE`来识别接口并完成配置的；如果设备没有对应的配置文件，则无法通过这两个命令启动或关闭
