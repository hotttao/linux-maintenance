# 24.5 iptables扩展匹配进阶
# iptables 显示扩展：
- 显示扩展: 必须显式指明使用的扩展模块进行的扩展；
- 使用帮助：
    - CentOS 6: `man iptables`
    - CentOS 7: `man iptables-extensions`
    - `rpm -ql iptables|grep '[[:lower:]]\+\.so$'`

## 1. iptables 可用显示扩展      
### 1.1 multiport扩展
- 作用: 以离散方式定义多端口匹配；最多指定15个端口；
- 参数:
    - [!] --source-ports,--sports port[,port|,port:port]...：指定多个源端口；
    - [!] --destination-ports,--dports port[,port|,port:port]...：指定多个目标端口；
    - [!] --ports port[,port|,port:port]...：指明多个端口；
```
> iptables -A INPUT -s 172.16.0.0/16 -d 172.16.100.67 -p tcp -m multiport --dports 22,80 -j ACCEPT

> iptables -A OUTPUT -s 172.16.100.67 -d  172.16.0.0/16 -p tcp -m multiport --sports 22,80 -j ACCEPT
```        

### 1.2 iprange扩展
- 作用: 指明连续的（但一般不能扩展为整个网络）ip地址范围；
- 参数:      
    - [!] --src-range from[-to]：源IP地址；
    - [!] --dst-range from[-to]：目标IP地址；

```            
> iptables -A INPUT -d 172.16.100.67 -p tcp --dport 80 -m iprange --src-range 172.16.100.5-172.16.100.10 -j DROP
> iptables -I INPUT -d 172.16.100.67 -p tcp -m multiport 22:23,80 -m iprange --src-range 172.16.100.1-172.168.100.120 -j ACCEPT
```        

### 1.3 string扩展
- 作用: 对报文中的应用层数据做字符串模式匹配检测；
- 参数:            
    - --algo {bm|kmp}：字符串匹配检测算法；
        - bm：Boyer-Moore
        - kmp：Knuth-Pratt-Morris
    - [!] --string pattern：要检测的字符串模式；
    - [!] --hex-string pattern：要检测的字符串模式，16进制格式；

```    
> iptables -A OUTPUT -s 172.16.100.67 -d 172.16.0.0/16 -p tcp --sport 80 -m string --algo bm --string "gay" -j REJECT
```

### 1.4 time扩展
- 作用: 根据将报文到达的时间与指定的时间范围进行匹配；
- 参数:
    - --datestart YYYY[-MM[-DD[Thh[:mm[:ss]]]]]
    - --datestop YYYY[-MM[-DD[Thh[:mm[:ss]]]]]
    - --timestart hh:mm[:ss]
    - --timestop hh:mm[:ss]
    - [!] --monthdays day[,day...]
    - [!] --weekdays day[,day...]
    - --kerneltz：使用内核上的时区，而非默认的UTC；
```
> iptables -A INPUT -s 172.16.0.0/16 -d 172.16.100.67 -p tcp --dport 80 -m time --timestart 14:30 --timestop 18:30 --weekdays Sat,Sun --kerneltz -j DROP
```

### 1.5 connlimit扩展
- 作用: 根据每客户端IP(也可以是地址块)做并发连接数数量匹配；
- 参数:
    - --connlimit-upto n：连接的数量小于等于n时匹配；
    - --connlimit-above n：连接的数量大于n时匹配；
```
> iptables -A INPUT -d 172.16.100.67 -p tcp --dport 21 -m connlimit --connlimit-above 2 -j REJECT
```

### 1.6 limit扩展
- 作用: 基于收发报文的速率做匹配；令牌桶过滤器；
- 参数:
    - --limit rate[/second|/minute|/hour|/day]
    - --limit-burst number    
```      
> iptables -I INPUT -d 172.16.100.67 -p icmp --icmp-type 8 -m limit --limit 3/minute --limit-burst 5 -j ACCEPT
> iptables -I INPUT 2 -p icmp -j REJECT
```        

## 2. state扩展
### 2.1 state扩展的功能
- 作用: 根据”连接追踪机制“去检查连接的状态；
- conntrack机制：追踪本机上的请求和响应之间的关系；状态有如下几种：
    - NEW：新发出请求；连接追踪模板中不存在此连接的相关信息条目，因此，将其识别为第一次发出的请求；
    - ESTABLISHED：NEW状态之后，连接追踪模板中为其建立的条目失效之前期间内所进行的通信状态；
    - RELATED：相关联的连接；如ftp协议中的数据连接与命令连接之间的关系；
    - INVALID：无效的连接；
    - UNTRACKED：未进行追踪的连接；
- 参数: [!] --state state
- 配置:
```    
> iptables -A INPUT -d 172.16.100.67 -p tcp -m multiport --dports 22,80 -m state --state NEW,ESTABLISHED -j ACCEPT
> iptables -A OUTPUT -s 172.16.100.67 -p tcp -m multiport --sports 22,80 -m state --state ESTABLISHED -j ACCEPT

> iptables -I OUTPUT -m state --state ESTABLISHED -j ACCEPT
```          
### 2.2 state 扩展相关配置
1. 调整连接追踪功能所能够容纳的最大连接数量：
    - /proc/sys/net/nf_contrack_max
    - 在一个非常繁忙的服务器上，一是要准备足够大的内存，二是将此配置调大
2. 已经追踪到到的并记录下来的连接：
    - /proc/net/nf_conntrack
3. 不同的协议的连接追踪时长：
    - /proc/sys/net/netfilter/

### 2.3 state 扩展相关问题          
iptables的链接跟踪表最大容量为/proc/sys/net/ipv4/ip_conntrack_max，
链接碰到各种状态的超时后就会从表中删除；当模板满载时，后续的连接可能会超时
解決方法一般有两个：
1. 加大 nf_conntrack_max 值
2. 降低 nf_conntrack timeout时间
```
# 加大 nf_conntrack_max 值
vi /etc/sysctl.conf
net.ipv4.nf_conntrack_max = 393216
net.ipv4.netfilter.nf_conntrack_max = 393216

# 降低 nf_conntrack timeout时间
vi /etc/sysctl.conf
net.ipv4.netfilter.nf_conntrack_tcp_timeout_established = 300
net.ipv4.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.ipv4.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.ipv4.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120

iptables -t nat -L -n
```

### 2.4 如何开放被动模式的ftp服务？
1. 装载ftp连接追踪的专用模块：
2. 放行命令连接(假设Server地址为172.16.100.67)：
3. 放行数据连接(假设Server地址为172.16.100.67)：
```
# 1. 装载ftp连接追踪的专用模块
> modinfo /lib/modules/3.10.0-514.el7.x86_64/kernel/net/netfilter/nf_conntrack_ftp
> modproble  nf_conntrack_ftp
> lsmod

# 2. 放行请求报文
#    命令连接: NEW, ESTABLISHED
#    数据连接: RELATED(仅数据连接的第一次连接建立), ESTABLISHED
> iptables -A INPUT -d 172.16.100.67 -p tcp --dport 21 -m state --state **NEW,ESTABLISHED** -j ACCEPT
> iptables -A OUTPUT -s 172.16.100.67 -p tcp --sport 21 -m state --state ESTABLISHED -j ACCEPT

# 3. 放行响应报文  ESTABLISHED
> iptables -A INPUT -d 172.16.100.67 -p tcp -m state --state **RELATED,ESTABLISHED** -j ACCEPT
> iptables -I OUTPUT -s 172.16.100.67 -p tcp -m state --state ESTABLISHED -j ACCEPT

# 4. 合并 iptables 规则
iptables -t filter -A INPUT -m --state ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -A INPUT -p tcp -d 192.168.1.108 -m multiport --dports 21,22,80 -m state --state NEW -j ACCEPT

iptable -t filter -A OUTPUT -m --state ESTABLISHED -j ACCEPT
```

## 3. iptables 规则优化和自定义
### 3.1 规则优化：
服务器端规则设定：任何不允许的访问，应该在请求到达时给予拒绝；
- 可安全放行所有入站的状态为ESTABLISHED状态的连接；
- 可安全放行所有出站的状态为ESTABLISHED状态的连接；
- 谨慎放行入站的新请求
- 有特殊目的限制访问功能，要于放行规则之前加以拒绝；

### 3.2 如何使用自定义链：
- 自定义链：需要被调用才能生效；自定义链最后需要定义返回规则；
- 返回规则使用的target叫做RETURN；

### 3.3 规则的保存及重载：
- 使用iptables命令定义的规则，手动删除之前，其生效期限为kernel存活期限；
- 保存规则：保存规则至指定的文件：
    - CentOS 6：
        - `service  iptables  save` :将规则保存至/etc/sysconfig/iptables文件中；
        - `iptables-save`  >  /PATH/TO/SOME_RULES_FILE
    - CentOS 7:
        - `iptables-save`  >  /PATH/TO/SOME_RULES_FILE
- 重载规则: 重新载入预存规则文件中规则：            
    - `iptables-restore` <  /PATH/FROM/SOME_RULES_FILE
    - CentOS 6：`service  iptables  restart`: 从 /etc/sysconfig/iptables文件中重载规则

### 3.4 Centos6 iptables 规则的开机自动载入
- chkconfig iptables on
- 脚本文件: `/etc/rc.d/init.d/iptables`
- 配置文件:
    - iptables data: `/etc/sysconfig/iptables`
    - iptables config: `/etc/sysconfig/iptables-config`
        - 选项: `IPTABLES_MODULE="`  " 配置要装载的模块
- 自动生效规则文件中的规则：
    - 用脚本保存各iptables命令；让此脚本开机后自动运行；
        - /etc/rc.d/rc.local文件中添加脚本路径 --- /PATH/TO/SOME_SCRIPT_FILE
    - 用规则文件保存各规则，开机时自动载入此规则文件中的规则；
        - /etc/rc.d/rc.local文件添加： iptables-restore < /PATH/FROM/IPTABLES_RULES_FILE

### 4. CentOS 7 新特性：
1. 引入了新的iptables前端管理服务工具 firewalld，包括很多管理工具，
    - firewalld-cmd
    - firewalld-config
2. 文档: http://www.ibm.com/developerworks/cn/linux/1507_caojh/index.html



### 5. 练习
练习：INPUT和OUTPUT默认策略为DROP；
```
1、限制本地主机的web服务器在周一不允许访问；新请求的速率不能超过100个每秒；web服务器包含了admin字符串的页面不允许访问；web服务器仅允许响应报文离开本机；
2、在工作时间，即周一到周五的8:30-18:00，开放本机的ftp服务给172.16.0.0网络中的主机访问；数据下载请求的次数每分钟不得超过5个；
3、开放本机的ssh服务给172.16.x.1-172.16.x.100中的主机，x为你的学号，新请求建立的速率一分钟不得超过2个；仅允许响应报文通过其服务端口离开本机；
4、拒绝TCP标志位全部为1及全部为0的报文访问本机；
5、允许本机ping别的主机；但不开放别的主机ping本机；
```

练习：判断下述规则的意义：
```
# iptables -N clean_in
# iptables -A clean_in -d 255.255.255.255 -p icmp -j DROP
# iptables -A clean_in -d 172.16.255.255 -p icmp -j DROP

# iptables -A clean_in -p tcp ! --syn -m state --state NEW -j DROP
# iptables -A clean_in -p tcp --tcp-flags ALL ALL -j DROP
# iptables -A clean_in -p tcp --tcp-flags ALL NONE -j DROP
# iptables -A clean_in -d 172.16.100.7 -j RETURN


# iptables -A INPUT -d 172.16.100.7 -j clean_in

# iptables -A INPUT  -i lo -j ACCEPT
# iptables -A OUTPUT -o lo -j ACCEPT


# iptables -A INPUT  -i eth0 -m multiport -p tcp --dports 53,113,135,137,139,445 -j DROP
# iptables -A INPUT  -i eth0 -m multiport -p udp --dports 53,113,135,137,139,445 -j DROP
# iptables -A INPUT  -i eth0 -p udp --dport 1026 -j DROP
# iptables -A INPUT  -i eth0 -m multiport -p tcp --dports 1433,4899 -j DROP

# iptables -A INPUT  -p icmp -m limit --limit 10/second -j ACCEPT
```
