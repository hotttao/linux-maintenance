# 32.1 dhcp服务简介
将一台主机接入互联网时，我们需要为其配置 `IP/Netmask`,`Gateway`,`DNS Server`。我们可以手动配置，也可以借助于 DHCP 协议实现自动化配置。

## 1. DHCP 简介
DHCP(Dynamic Host Configuration Protocol) 动态主机配置协议

### 1.1 演化历史
dhcp 的前身是 `bootp` 出现于无盘工作站，: boot protocol --> dhcp
  租约：
    2hours:
      50%: 1hours --> 2hours
        50%：1hours --> 2hours
          75%: 0.5hours --> 2hours
            87.5%: 0.25hours --> 2hours

      dhcp discover

### 1.2 工作过程
  1、Client: dhcp discover：发现
  2、Server: dhcp offer：(IP/netmask, gw)
  3、Client：dhcp request
  4、Server: dhcp ack

  续租：
    Client: dhcp request
    Server: dhcp ack

    Server: dhcp nak

## 2. dhcp 服务
Linux DHCP协议的实现程序：dhcp, dnsmasq
### 2.1 程序组成
```
# rpm -ql dhcp|egrep -v "(share|man)"
/etc/NetworkManager
/etc/NetworkManager/dispatcher.d
/etc/NetworkManager/dispatcher.d/12-dhcpd
/etc/dhcp/dhcpd.conf              # ipv4 的配置文件
/etc/dhcp/dhcpd6.conf             # ipv6 地址分配相关的配置文件
/etc/dhcp/scripts
/etc/dhcp/scripts/README.scripts
/etc/openldap/schema/dhcp.schema
/etc/sysconfig/dhcpd
/usr/bin/omshell
/usr/lib/systemd/system/dhcpd.service      # ipv4 unit file
/usr/lib/systemd/system/dhcpd6.service     # ipv6 unit file
/usr/lib/systemd/system/dhcrelay.service
/usr/sbin/dhcpd                   # dhcp 服务的主程序
/usr/sbin/dhcrelay                # dhcp 中继服务的主程序
/var/lib/dhcpd
/var/lib/dhcpd/dhcpd.leases       # 已经分配的的 IP 地址的相关信息
/var/lib/dhcpd/dhcpd6.leases
```

需要注意的是 dhcp 的服务器端监听在 udp 的 67 号端口，dhclient 则监听在 `68/udp`

### 2.2 服务配置
dhcp 的 rpm 包提供了一个dhcp 配置的参考文件 `/usr/share/doc/dhcp-4.2.5/dhcpd.conf.example`，我们拿过来直接使用

```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "example.org";                            # 搜索域
option domain-name-servers ns1.example.org, ns2.example.org; # DNS 服务器地址

default-lease-time 600;    # 默认租约期限
max-lease-time 7200;       # 最大租约期限
log-facility local7;

subnet 10.254.239.32 netmask 255.255.255.224 {
  range  10.254.239.40 10.254.239.60;  
  option broadcast-address 10.254.239.31;    # 广播地址
  option routers rtr-239-32-1.example.org;   # 默认网关
  filename "pxelinux.0";
  next-server 172.16.100.67;
}


host fantasia {
  hardware ethernet 08:00:07:26:c0:a5;
  fixed-address 172.16.100.6;
}

```

配置:
1. 配置段:
  1. subnet 域: 配置可动态分配的地址范围
  2. host 域: 为特定主机绑定地址
2. 配置选项: 可以放置在配置内，也可以放置在配置段中
  3. filename: 指明引导文件名称；
  4. next-server：提供引导文件的服务器IP地址；

### 2.3 dhclient
dhclient 是 dhcp 客户端程序

`dhclient options`
- 作用:
- 选项:
  - `-d`: 将 dhclient 工作于前台，显示 dhcp 的工作过程
