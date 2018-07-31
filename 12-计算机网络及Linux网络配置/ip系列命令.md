# 12.4 网络属性配置之 ip 系列命令


## 1. iproute家族：
### 1.1 ip命令：
- 作用: show / manipulate routing, devices, policy routing and tunnels
- 命令: ip [ OPTIONS ] OBJECT { COMMAND | help }
    - OBJECT := { link | addr | route | netns  }
    - 注意： OBJECT可简写，各OBJECT的子命令也可简写；

#### ip link
ip link： network device configuration
1. ip link set:
    - 作用: 更改网络接口属性
    - 参数：
        - dev NAME (default) {up|down}：指明要管理的设备，dev关键字可省略，up 表示启用，down 表示关闭；
        - multicast on或multicast off：启用或禁用多播功能；
        - name NAME：重命名接口
        - mtu NUMBER：设置MTU的大小，默认为1500；
        - netns PID：ns为namespace，用于将接口移动到指定的网络名称空间；        
2. ip link show
    - 作用: 显示网络接口属性
    - 参数:
        - [dev NAME] (default)：指明要显示的接口，dev关键字可省略；
        - [up]: 仅仅显示启用状态的接口设备
3. ip  link  help -  显示简要使用帮助；
```
ip link  set dev  ens33  up  # 启用接口
ip link show  #
```

#### ip netns        
ip netns：  - manage network namespaces.      
- ip  netns  list：列出所有的netns
- ip  netns  add  NAME：创建指定的netns
- ip  netns  del  NAME：删除指定的netns
- ip  netns  exec  NAME  COMMAND：在指定的netns中运行命令

#### ip address
ip address - protocol address management.            
1. ip addr  {add | del}  IP  dev  IFACE  IFADDR  
    - add: 为网络接口添加 IP 地址
    - del: 删除网络接口的 IP 地址
    - [dev] IFACE: 指定网络接口
    - IFADDR
        - [label NAME]：为额外添加的地址指明接口别名；
        - [broadcast ADDRESS]：广播地址；会根据IP和NETMASK自动计算得到；
        - [scope SCOPE_VALUE]：指明网络接口的作用域
            - global：全局可用；
            - link：接口可用；
            - host：仅本机可用；                                      
3. ip addr show
    - 作用: 显示网络接口地址详细信息
    - 参数:
        - [dev IFACE]: 显示特定接口
        - [label PATTERN]: 显示指定模式别名的接口
4. ip addr list  [IFACE]：显示接口的地址；
5. ip addr flush  dev  IFACE
    - 作用: 清空网络设备的IP地址
    - 参数:
        - [label PATTERN]: 删除指定模式别名的接口

#### ip route
routing table management
1. ip  route  add  TARGET via GW  [dev  IFACE]  [src SOURCE_IP]
    - TARGET:
        - 主机路由: IP
        - 网络路由: NETWORK/MASK
    - src:  一个网卡存在多个地址时，指定出口IP
    - eg:
        - `ip route add 192.168.0.0/24  via 10.0.0.1  dev eth1 src  10.0.20.100`
        - `ip  route  add default  via  GW`                 
2. ip route change - change route
3. ip route replace - change or add new one                   
5. ip  route  del  TARGET
    - `ip  route del  192.168.1.0/24`
6. ip route show
    - 参数:
        - [dev IFACE]: 查看网卡
        - [via PREFIX]: 查看特定网关的路由
7. ip route flush
    - 参数: 同 ip route show
8. ip  route  get
    - 作用: 获取一条特定的路由信息
    - eg: `ip route  get  192.168.0.0/24`

## 2. ss命令：
ss  [options]  [ FILTER ]
- 作用：类似于 netstat
- 选项：
    - -t：TCP协议的相关连接
    - -u：UDP相关的连接
    - -w：raw socket相关的连接
    - -x：unix socket 相关
    - -l：监听状态的连接
    - -a：所有状态的连接
    - -n：数字格式
    - -p：相关的程序及其PID
    - -e：扩展格式信息
    - -m：内存用量
    - -o：计时器信息
- FILTER := [ state TCP-STATE ]  [ EXPRESSION ]
    - TCP-STATE：
        - LISTEN：监听
        - ESTABLISEHD：建立的连接
        - FIN_WAIT_1：
        - FIN_WAIT_2：
        - SYN_SENT：
        - SYN_RECV：
        - CLOSED：          
    - EXPRESSION：
        - dport = 目标端口号
        - sport = 源端口号
        - 示例：'( dport = :22 or sport = :22)'
-eg:
    - ss -tan  '(  dport = :22 or sport = :22  )'
    - ss -tan  state  ESTABLISHED
