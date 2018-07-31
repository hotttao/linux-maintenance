# 12.6 网络属性配置之 nmcli 系列命令
## 1. nmcli命令：
nmcli  [ OPTIONS ] OBJECT { COMMAND | help }
OBJECT:
1. device
    - 作用: 显示和管理网络接口，类似 ip link
    - COMMAND := { status | show | connect | disconnect | delete | wifi | wimax }
2. connection
    - 作用: 启动，停止，管理网络连接，类似 ip addr
    - COMMAND := { show | up | down | add | edit | modify | delete | reload | load }
3. modify [ id | uuid | path ] <ID> [+|-]<setting>.<property> <value>

### 1.1  如何修改IP地址等属性：
nmcli  connection  modify  IFACE  [+|-]setting.property  value
1. 作用: 直接修改 /etc/sysconfig/network-script/ifcfg-IFACE 文件，修改完成不会生效，需要重启
1. IFACE: 接口标识
2. setting.property
    - ipv4.address
    - ipv4.gateway
    - ipv4.dns1
    - ipv4.method
        - manual: 手动配置
        - dhcp:
```
localectl  list-locales
localectl set-locale LANG=en_US.utf8
nmcli g status # 显示网络接口状态
nmcli device show ens33
nmcli connection  modify  ens33  ipv4.address 192.168.1.101
# 重启以生效修改
nmcli con down ens33; nmcli  connection up ens33
```
### 1.2 nmtui
nmcli 命令的图形化工具
