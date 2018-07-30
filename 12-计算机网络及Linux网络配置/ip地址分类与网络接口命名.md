# 12.2  ip地址分类与网络接口命名
## 1. IP地址分类
通过上一节我们知道，IP 地址用于标识网络及网络中的主机，IPv4 由 32 位字节表示，那到底多少字节表示网络，多少字节表示网络中的主机呢？ 按照用于表示网络的字节数将 IP 地址分为 ABCDE 五大类:
1. A类：
    - 第一段为网络号，后三段为主机号
    - 网络号：0 000 0000 - 0 111 1111：1-127
    - 网络数量：126，127
    - 每个网络中的主机数量：2^24-2
    - 默认子网掩码：255.0.0.0，/8
    - 私网地址：10.0.0.0/8
2. B类：
    - 前两段为网络号，后两段为主机号
    - 网络号：10 00 0000 - 10 11 1111：128-191
    - 网络数：2^14
    - 每个网络中的主机数量：2^16-2
    - 默认子网掩码：255.255.0.0，/16
    - 私网地址：172.16.0.0/16-172.31.0.0/16                                
3. C类：
    - 前三段为网络号，最后一段为主机号
    - 网络号：110 0 0000 - 110 1 1111：192-223
    - 网络数：2^21
    - 每个网络中的主机数量：2^8-2
    - 默认子网掩码：255.255.255.0,  /24
    - 私网地址: 192.168.0.0/24-192.168.255.0/24
4. D类：组播
    - 网络号: 1110 0000 - 1110 1111：224-239
5. E类：科研
    - 网络号: 1111 0000 - 1111 1111: 240-255


## 2. 网络配置方式
将Linux主机接入到网络中：
1. IP/NETMASK：本地通信
2. 路由（网关）：跨网络通信
3. DNS服务器地址：基于主机名的通信
    - 主DNS服务器地址
    - 备用DNS服务器地址
    - 第三备份DNS服务器地址

### 2.1 静态指定：
1. ifcfg家族：
    - `ifconfig`：配置IP，NETMASK
    - `route`：路由
    - `netstat`：状态及统计数据查看
2. iproute2家族：
    - `ip OBJECT`：
        - addr：地址和掩码；
        - link：接口
        - route：路由
    - ss：状态及统计数据查看
3. CentOS 7：nm(Network Manager)家族
    - `nmcli`：命令行工具
    - `nmtui`：text window 工具
4. Centos6:
    - `system-config-network-tui`
    - `setup`          
5. 配置文件：
    - RedHat及相关发行版:`/etc/sysconfig/network-scripts/ifcfg-NETCARD_NAME`
    - `system-config-netword-tui(setup)``
4. 注意：
    - DNS服务器指定: 配置文件：`/etc/resolv.conf`
    - 本地主机名配置
        - hostname, 配置文件：`/etc/sysconfig/network`
        - CentOS 7：`hostnamectl`

### 2.2 动态分配：
- 方式: 依赖于本地网络中有DHCP服务
- DHCP: Dynamic Host Configure Procotol

## 3. 网络接口命名方式：
### 3.1 命名机制
1. 传统命名(centos6)：
    - 以太网：ethX, [0,oo)，例如eth0, eth1, ...
    - PPP网络：pppX, [0,...], 例如，ppp0, ppp1, ...    
2. 可预测命名方案(CentOS7)：支持多种不同的命名机制
    1. 如果Firmware或BIOS为主板上集成的设备提供的索引信息可用，则根据此索引进行命名，如eno1, eno2, ...  -- Fireware(固件)
    2. 如果Firmware或BIOS为PCI-E扩展槽所提供的索引信息可用，且可预测，则根据此索引进行命名，如ens1, ens2, ...
    3. 如果硬件接口的物理位置信息可用，则根据此信息命名，如enp2s0, ...
    4. 如果用户显式定义根据MAC地址命名，例如enx122161ab2e10, ...
    5. 上述均不可用，则仍使用传统方式命名；
    - 附注: 上述命名机制中，有的需要 biosdevname 程序的参与

### 3.2 名称组成格式  
1. 前缀
    - en：ethernet 以太网接口
    - wl：wlan 无线局域网设备
    - ww：wwan 无线广域网设备
2. 后缀
    - o<index>：集成设备的设备索引号；(onbus)
    - s: slot扩展槽的索引号；
    - x: 基于MAC地址的命名；
    - p: bus-s-slot  基于总线及槽的拓扑结构进行命名；
        - bus: PCI 总线编号
        - slot: 总线上的扩展槽编号
        - eg: p2s1

### 3.3 网卡设备的命名过程
1. udev 辅助工具程序 /lib/udev/rename_device 会根据 /usr/lib/udev/rules.d/60-net.rules 中的指示去查询  /etc/sysconfig/network-script/ifcfg-IFACE 配置文件，根据HWADDR 读取设备名称
2. biosdevname 根据 /user/lib/udev/rules.d/71-boosdevname.rules
3. 通过检查网络接口设备，根据 /usr/lib/udev/rules.d/75-net-description 中 ID_NET_NAME_ONBOARD 和 ID_NET_NAME_SLOT,ID_NET_NAME_PATH 命名

### 3.4 Centos7 回归传统方式命名步骤
1. vim /etc/default/grub 配置文件，添加 GRUB_CMDLINE_LINUX="net.ifnames=0 "
2. 为 grub2 生成配置文件
    - grub2-mkconfig -o /etc/grub2.cfg
3. 重启系统
