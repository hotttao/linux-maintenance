# 12.4 网络属性配置之 ip 系列命令
本节我们来介绍 Linux 网络属性配置的第二组系列命令 ip 命令。ip 命令是 Linux 的"新贵"，相比于 ifcfg 它们更加的高效。ip 系列包括如下几个命令
1. ip 命令: 有众多子命令，拥有配置网络地址，网络接口属性，路由等多种功 能
2. ss 命令: netstat 命令的优化版，用于查看网络连接状态

## 1. ip
`ip [ OPTIONS ] OBJECT { COMMAND | help }`
- 作用: show / manipulate routing, devices, policy routing and tunnels.
- `OBJECT`: 作用对象
    - `link`: 网络接口属性
    - `addr`: 网络地址
    - `route`: 路由
    - `netns`: 网络命名空间
- `COMMAND`: 每个作用对象上的可用子命令
- `help`: 是一个通用子命令，用于显示特定作用对象的可用命令
- 注意： OBJECT可简写，各OBJECT的子命令也可简写；

通过上面的展示可以看出，ip 将网络地址，路由，网络接口等划分成了一个个独立的对象，每个独立的对象拥有特定的子命令来对其进行管理和配置。

### 1.1 ip link
ip link： network device configuration，是用来配置网络设备的子命令，可用于管理网络接口的各种属性。其常用子命令如下

#### ip link set
`ip link set [dev] NAME options`
- 作用: 更改网络接口属性
- `[dev] NAME`:指明要管理的设备，dev关键字可省略
- 参数：
    - `up|down`：，up 表示启用，down 表示关闭；
    - `multicast on|off`：启用或禁用多播功能；
    - `name NAME`：重命名接口
    - `mtu NUMBER`：设置MTU的大小，以太网默认为1500；
    - `netns PID`：ns为namespace，用于将接口移动到指定的网络名称空间

```
ip link set eth1 down
ip link set multicast on

ip link show

# 改名
ip link set eno1 down
ip link set name eno8888
ip link set eno8888 up
```

#### ip link show|list
`ip link show|list options`
- 作用: 显示网络接口属性
- 参数:
    - `[dev] NAME`：指明要显示的接口，dev关键字可省略；
    - `up`: 仅仅显示启用状态的接口设备

#### ip link help
`ip link help`
- 作用: 显示 ip link 简要使用帮助


### 1.2 ip netns        
ip netns：manage network namespaces. 用于管理网络命名空间。网络命名空间在虚拟化中具有重要作用，我们会在虚拟化重新介绍 ip netns 的使用，此处仅作了解即可

`ip netns COMMAND`:      
- `ip  netns  list`：列出所有的netns
- `ip  netns  add  NAME`：创建指定的netns
- `ip  netns  del  NAME`：删除指定的netns
- `ip  netns  exec  NAME  IP_COMMAND`
    - 作用: 在指定的netns中运行命令
    - NAME: 表示指定的命名空间
    - IP_COMMAND: 任何可使用的 ip 命令


```
ip netns help

ip netns add mynet
ip link set eno1 netns mynet  # 将 eno1 添加到 mynet 名称空间中
ip link show

ip netns exec mynet ip link show # 显示 mynet 中的网络设备

ip netns del mynet

ip link show
```

### 1.3 ip address
ip address:protocol address management. 用于管理网络地址，作用类似于 ifconfig 命令

#### ip addr add  
`ip addr add IP  dev IFACE IFADDR`  
- 作用: 为网络接口添加 IP 地址
- 参数:
    - `IP`: ip/netmask
    - `dev IFACE`: 指定网络接口
- IFADDR: 为地址的添加的额外属性
    - `label NAME`：为额外添加的地址指明接口别名；
    - `broadcast ADDRESS`：广播地址；会根据IP和NETMASK自动计算得到，一般无需指明
    - `scope SCOPE_VALUE`：指明网络接口的作用域了解即可
        - `global`：全局可用；
        - `link`：接口可用；
        - `host`：仅本机可用；  

```
ifconfig eth1 0

# 一个网卡可添加多个地址
ip addr add 192.168.1.101/24 dev eth1
ip addr add 192.168.100.100/24 dev eth1 label eth1:0  # 指定接口别名
```

#### ip addr delete
`ip addr add IP  dev IFACE`
- 作用: 删除网络接口的 IP 地址，delete 可简写成 del
- 参数:
    - `IP`: 指明要删除的 ip/netmask
    - `dev IFACE`: 指定网络接口

```
ip addr del 192.168.1.101/24 dev eth1
```


#### ip addr show
`ip addr show|list options`
- 作用: 显示网络接口地址详细信息
- 选项:
    - `[dev] IFACE`: 显示特定接口的地址，默认显示所有接口
    - `label PATTERN`: 显示指定模式别名的接口

#### ip addr flush
`ip addr flush  dev IFACE options`
- 作用: 清空网络设备的IP地址，不支持简写
- 选项:
    - `label PATTERN`: 删除指定模式别名的接口

```
ip addr flush dev eth1
ip addr show eth1
```

### 1.4 ip route
ip route routing table management，路由表管理

#### ip route add  
`ip route {add|change|replace} TARGET via GW  options`
- 作用: add,change,replace 使用方式类似
    - add: 添加路由条目，可根据路由信息自动判断是主机路由还是网络路由
    - change: 更改路由
    - replace: 更改或添加路由    
- 参数:
    - `TARGET`: 主机路由即 IP，网络路由即 NETWORK/MASK
    - `via GW`: 网关或路由的下一跳
- 选项:
    - `dev  IFACE`: 指明从哪个网络接口发送报文
    - `src SOURCE_IP`: dev 指明的网卡存在多个地址时，指定出口IP

```
ip addr add 10.0.0.100/8 dev eth1
ip addr add 10.0.20.100/8 dev eth1

ip route add 192.168.0.0/24  via 10.0.0.1  dev eth1 src  10.0.20.100
ip  route add default  via  192.168.1.1`                 
```

#### ip  route  delete  
`ip route delete TARGET [via GW]`
- 作用: 删除路由条目
- 参数:
    - `TARGET`: 主机路由即 IP，网络路由即 NETWORK/MASK
    - `[via GW]`: 如果 TARGET 能唯一指明路由条目则无需指明网管
- eg: `ip  route del  192.168.1.0/24`


#### ip route show
`ip route show options`
- 作用: 查看路由条目
- 选项:
    - `TARGET`: 显示特定路由条目
    - `[dev] IFACE`: 查看特定网卡的路由条目
    - `via PREFIX`: 查看特定网关的路由条目

#### ip route flush
`ip route flush options`
- 作用: 清空路由条目
- 选项:
    - `prefix/mask`: 删除特定前缀的路由条目
    - `[dev] IFACE`: 查看网卡的路由条目
    - `via PREFIX`: 删除特定网关的路由

```
ip route flush 10/8  # 删除前缀为 10，掩码为 8 位的条目
ip route flush 192/8  # 指定的 192/8 无法删除 192/24 的路由条目
```

#### ip route get
`ip route get TARGET [via GW]`
- 作用: 获取一条特定的路由信息

```
ip route  get  192.168.0.0/24
```

## 2. ss命令：
`ss  [options]  [ FILTER ]`
- 作用：类似于 netstat，用于查看网络连接状态
- 类 netstat 选项：
    - `-t`：TCP协议的相关连接
    - `-u`：UDP相关的连接
    - `-w`：raw socket相关的连接
    - `-x`：unix socket 相关
    - `-l`：监听状态的连接
    - `-a`：所有状态的连接
    - `-n`：数字格式
    - `-p`：相关的程序及其PID
    - `-e`：扩展格式信息
- 特有选项:
    - `-m`：内存用量
    - `-o`：计时器信息
- FILTER: ss 用于筛选的表达式
    - 格式: `[ state TCP-STATE ]  [ EXPRESSION ]`
    - TCP-STATE：
        - `LISTEN`：监听
        - `ESTABLISEHD`：建立的连接
        - `FIN_WAIT_1`：
        - `FIN_WAIT_2`：
        - `SYN_SENT`：
        - `SYN_RECV`：
        - `CLOSED`：          
    - EXPRESSION：
        - `dport = 目标端口号`
        - `sport = 源端口号`
        - eg：`'( dport = :22 or sport = :22 )'` - 每个部分都必须有空格隔开

```
ss -tan  '(  dport = :22 or sport = :22  )'
ss -tan  state  ESTABLISHED
```
