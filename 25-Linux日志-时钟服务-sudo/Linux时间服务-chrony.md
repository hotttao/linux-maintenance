# 25.1 Linux时间服务-chrony
Linux 中很多服务在跨主机通信需要保持时间同步，特别是对于一个集群而言，多台主机之间的时间必需严格一致。Linux 在启动时会从硬件中读取时间，在操作系统启动之后，系统时钟就会同硬件时钟相独立。系统始终依赖 CPU 的工作频率更新时间，因此不同主机之间的时间很难保持一致。特别是在虚拟化环境中，每个虚拟机只能获取一部份的 CPU 周期，时间基本不可能保持一致，此时就需要进行时间同步。本节我们就来 Linux 中的时间服务，内容包括:
1. Linux 中时间同步的协议和方式
2. ntp 和 chrony

## 1. Linux 中的时间同步
### 1.1 Linux 时间同步的方式
ntp(Network Time Protocol) 是 Linux 同步时间的协议。当时间出现偏差时，Linux 不能将当前时间直接调整为准确时间，这是因为 Linux 上很多服务依赖于时间的连续性，时间不能跳越。Linux 只能通过让时间走"更快"或"更慢"来同步时间。

ntp 协议的最早实现是 ntp 服务，但是 ntp 存在一个缺陷，时间同步太慢，在时间相差较大时需要很长时间才能完成时间同步。chrony 服务是 ntp 的改进版本，它采用了一种很精巧的策略，在保证时间连续的同时能在几秒甚至更短的时间内完成时间同步。

### 1.2 Linux 时间同步服务
ntp，chrony 既是客户端也是服务端，这是因为我们的系统需要实时同步时间，因此 ntp，chrony 要作为后台进程实时进行。ntp 和 chrony 都监听在 utp的 123 端口上，因此 chrony 是兼容 ntp 的，即 ntp 的客户端也能从 chrony 服务同步时间。

### 1.3 时间同步配置
在 Linux 中同步时间只需要，安装 chrony，修改其配置文件，更改其同步时间服务器，如果其同时作为服务器使用，需要修改 `allow` 参数指定允许按些客户端过来同步；配置完成后启动 chrony 的服务，并将其设置为开机自动启动即可。

也可以使用 ntpdate 命令临时进行手动时间同步。下面我们就来详细介绍 ntp 和 chrony 的配置。Linux 上建议使用 chrony。

```
# 1. 安装 chrony 服务
yum install chrony

# 2. 修改配置文件，详细配置见下
vim /etc/chrony.conf
allow=           # 允许同步的客户端
server=          # 时间同步服务器

# 3. 启动服务
systemctl start chronyd
systemctl enable chronyd
```

## 2. ntp
```
# rpm -ql ntp
/etc/dhcp/dhclient.d
/etc/dhcp/dhclient.d/ntp.sh
/etc/ntp.conf            # 配置文件
/etc/ntp/crypto
/etc/ntp/crypto/pw
/etc/sysconfig/ntpd
/usr/bin/ntpstat
/usr/lib/systemd/ntp-units.d/60-ntpd.list
/usr/lib/systemd/system/ntpd.service
/usr/sbin/ntp-keygen
/usr/sbin/ntpd
/usr/sbin/ntpdc
/usr/sbin/ntpq
/usr/sbin/ntptime
/usr/sbin/tickadj
```

### 2.1 配置文件
```
# cat /etc/ntp.conf |grep -v "^#"
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1                  # 允许哪些主机过来同步时间
restrict ::1
server 0.centos.pool.ntp.org iburst # 时间服务器地址
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

### 2.2 时间同步命令
`ntpdate server_ip`
- 作用: 手动向 server_ip 指向的服务同步时间


## 3. chrony
```
# rpm -ql chrony
/etc/NetworkManager/dispatcher.d/20-chrony
/etc/chrony.conf               # chrony 配置文件
/etc/chrony.keys
/etc/dhcp/dhclient.d/chrony.sh
/etc/logrotate.d/chrony
/etc/sysconfig/chronyd         
/usr/bin/chronyc               # chrony 服务管理工具
/usr/lib/systemd/ntp-units.d/50-chronyd.list
/usr/lib/systemd/system/chrony-dnssrv@.service
/usr/lib/systemd/system/chrony-dnssrv@.timer
/usr/lib/systemd/system/chrony-wait.service
/usr/lib/systemd/system/chronyd.service
/usr/libexec/chrony-helper
/usr/sbin/chronyd              # 客户端亦是服务端程序
```

### 3.1 组成
chrony 由如下几个部分组成:
- 配置文件：`/etc/chrony.conf`
- 主程序文件：`chronyd`
- 工具程序：`chronyc`
- unit file: `chronyd.service`

### 3.2 配置
```bash
$ cat /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
#local stratum 10

# Specify file containing keys for NTP authentication.
#keyfile /etc/chrony.keys

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking
```

核心配置选项包括:
- `server`:指明时间服务器地址，本机会向 server指向的机器同步时间
- `allow NETADD/NETMASK`: chrony 作为服务端使用时，允许哪些网络的主机同步时间
- `allow all`:允许所有客户端主机
- `deny NETADDR/NETMASK`: 不允许哪些网络的主机同步时间
- `deny all`:拒绝所有客户端
- `bindcmdaddress`:命令管理接口监听的地址, `chronc` 命令连接此地址对 chrony进行远程管理，因此不要监听在公网地址上
- `local stratum 10`:即使自己未能通过网络时间服务器同步到时间，也允许将本地时间作为标准时间授时给其它客户端


### 3.3 chronc
chronc 是 chrony 服务的管理工具，它能远程连接 chrony 服务，chrony 会监听在 `bindcmdaddress` 参数配置的地址，等待 chronc 连接。chronc 是一个交互的客户端工具，最常使用的子命令为 `sources`，`sourcestats`，`sourcestats -v`，`help`。

```
# chronyc
chronyc> sources
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? ns3106355.ip-37-187-100.>     2   7     5    14    +20ms[  +22ms] +/-  236ms
^? .                             0   8     0     -     +0ns[   +0ns] +/-    0ns
^? cn.ntp.faelix.net             0   8     0     -     +0ns[   +0ns] +/-    0ns
^* ntp2.flashdance.cx            2   6    45    15  +3857us[+6412us] +/-  246ms

chronyc> sourcestats
chronyc> sourcestats -v
chronyc> help
```
