# 12.6 网络属性配置之 nmcli 系列命令
本节我们来介绍 Linux 网络属性配置的第三组系列命令 nm。nm(network management) 是 Centos7 新增的命令，使用方式类似 ip 命令，将网络属性分成了多个 OBJECT，每个 OBJECT 都有众多子命令用于对其进行管理配置。nm 主要包含两个工具:
1. nmcli: nm 的命令行工具
2. nmtui: nm 的图形客户端

## 1. nmcli
`nmcli  [ OPTIONS ] OBJECT { COMMAND | help }`
- OBJECT:
  - device: 显示和管理网络接口，类似 ip link
  - connection: 启动，停止，管理网络连接，类似 ip addr

### 1.1 nmcli device
`nmcli device COMMAND`
- COMMAND:`{status | show | connect | disconnect | delete | wifi | wimax }`

### 1.2 nmcli connection
`nmcli connection COMMAND`
- COMMAND: `{ show | up | down | add | edit | modify | delete | reload | load }`

#### nmcli connection modify
`nmcli connection modify IFACE [+|-]<setting>.<property> <value>`
- 作用: 如何修改IP地址等属性：
- 效力: 直接修改 ifcfg-IFACE 文件，修改完成不会生效，需要重启
- 参数:
  - IFACE: 接口标识
  - setting.property: 网络属性值
    - ipv4.address
    - ipv4.gateway
    - ipv4.dns1
    - ipv4.method
        - manual: 手动配置
        - dhcp

```
localectl  list-locales
localectl set-locale LANG=en_US.utf8
nmcli g status # 显示网络接口状态
nmcli device show ens33
nmcli connection  modify  ens33  ipv4.address 192.168.1.101
# 重启以生效修改
nmcli con down ens33; nmcli  connection up ens33
```

## 2. nmtui
nmcli 命令的图形化工具
