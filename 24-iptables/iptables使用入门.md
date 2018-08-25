# 24.2 iptables使用入门
iptables 是防火墙规则的管理工具，使用复杂，但是也有规律可循，个人总结起来就是"在哪，对什么报文，作出什么样的处理"。
1. iptables 有四表五链，首先要确定在iptables 的哪个表，哪个链条添加规则，这取决于要实现的功能和报文的流经途径
2. 规则添加就是要对我们要处理的报文作出处理，因此就要指定报文的匹配条件和处理动作。
个人觉得按照这样的逻辑去记忆能比较容易记住 iptables 的使用方式。下面我们就来详细介绍 iptables 的使用

## 1. iptables 命令简介
### 1.1 规则配置原则
iptables 会按照链上的规则每次从上至下依次检查，如果报文匹配并作出处理，那么就不会继续匹配接下来的规则。因此这样的检查次序隐含一定的应用法则：
1. 同类规则（访问同一应用），匹配范围小的放上面；
2. 不同类的规则（访问不同应用），匹配到报文频率较大的放在上面；
3. 将那些可由一条规则描述的多个规则合并起来；
4. 设置默认策略；
这些应用原则其实很好懂，将频率高的放在最上面是为了减少匹配的次数，匹配范围小的放上面是为了让范围小的规则优先生效，很容易明白。

### 1.2 命令组成
```
iptables  
1. [-t table]  COMMAND  chain          -- 基本命令          
2. [-m matchname [per-match-options]]  -- 匹配条件  
3. -j targetname [per-target-options]  -- 处理动作  
```
iptables 命令组成如上所示:
1. `-t`: 指定操作的表，默认为 `filter`
2. `chain`: 指定操作的链
3. `[-m matchname [per-match-options]]`: 指定匹配条件，根据协议报文特征进行报文筛选，分为
    - 基本匹配条件: iptables 内置，不会使用扩展模块，不需要 `-m` 选项指定模块
    - 扩展匹配条件: 需要使用 `-m` 选项指定使用的扩展模块
4. `-j targetname [per-target-options]`: 指定处理动作，可分为:
    - 基本处理动作: iptables 内置的处理动作
    - 扩展处理动作: 通过扩展模块扩展的处理动作
    - 自定义处理机制: 通常指的是自定义链

### 1.3 iptables 的扩展机制
iptables 是高度模块化的，可以通过扩展模块更细粒度设置匹配条件和处理动作，这就是上面所说的扩展匹配条件和扩展处理动作。

```
$ rpm -ql iptables |grep xtables
/usr/lib64/xtables/libip6t_MASQUERADE.so  #  IPv6 的扩展模块
/usr/lib64/xtables/libip6t_REDIRECT.so
/usr/lib64/xtables/libip6t_REJECT.so
/usr/lib64/xtables/libip6t_SNAT.so
.....
/usr/lib64/xtables/libip6t_ah.so
/usr/lib64/xtables/libip6t_dst.so
/usr/lib64/xtables/libip6t_eui64.so
.....
/usr/lib64/xtables/libipt_MASQUERADE.so   # IPv4 扩展模块
/usr/lib64/xtables/libipt_MIRROR.so
......
/usr/lib64/xtables/libxt_CT.so    # libxt
/usr/lib64/xtables/libxt_DSCP.so
/usr/lib64/xtables/libxt_HMARK.so
.....
```
iptables 扩展模块的命名机制:
1. 小写子母命名的是匹配条件
2. 大写子母命令的是处理动作
3. libip6t 开头的对应于 IPv6 协议
4. libxt 和 libxt 开头的对应于 IPv4 协议

### 1.4 iptables 自定义链
iptables的链分为内置链和自定义链
- 内置链：对应于netfilter 的勾子函数(hook function)
- 自定义链接：用于内置链的扩展和补充，可实现更灵活的规则管理机制；但报文不会经过自定义链，只能在内置链上通过规则进行引用后生效

有了这些基本介绍，我们现在开始介绍 iptables 的详细使用。

## 2. iptables 命令使用
```
iptables  
1. [-t table]  COMMAND  chain          -- 基本命令          
2. [-m matchname [per-match-options]]  -- 匹配条件  
3. -j targetname [per-target-options]  -- 处理动作  
```

### 2.1 基本命令    
`iptables  [-t table]  COMMAND  chain`
- `-t`: 指定操作的表，默认为 `filter` (`raw`, `mangle`, `nat`, `filter`)
- `chain`: 指定操作的链(`PREROUTING`，`INPUT`，`FORWARD`，`OUTPUT`，`POSTROUTING`)
- COMMAND: 字命令
    - 链管理：
        - `-N`：new, 自定义一条新的规则链；
        - `-X`：delete，删除自定义的空的规则链，链非空或被其他链引用无法删除
        - `-Z`：zero，将规则计数器置零；
        - `-F`：flush，清空指定的规则链；省略表示清空指定表上的所有链
        - `-P`：Policy，为指定链设置默认策略；对filter表中的链而言，其默认策略有：
            - `ACCEPT`：接受
            - `DROP`：丢弃
            - `REJECT`：拒绝
        - `-E`：重命名自定义链；引用计数不为0的自定义链不能够被重命名，也不能被删除；
    - 规则管理：
        - `-A`：append，追加；
        - `-I`：insert, 插入，要指明位置，省略时表示第一条；
        - `-D`：delete，删除；
            - 指明规则序号；
            - 指明匹配条件；
        - `-R`：replace，替换指定链上的指定规则；                       
    - 查看：
        - `-L`：list, 列出指定鏈上的所有规则；
          - `-n`：numberic，以数字格式显示地址和端口号(不对ip地址进行反解)；
          - `-v`：verbose，详细信息，还有`-vv`, `-vvv`选项显示更详细的信息
          - `-x`：exactly，显示计数器结果的精确值；
          - `--line-numbers`：显示规则的序号；
          - 注意: `-nvL` 可以，`-Lnv` 不可以，因为 `-L` 是命令，其他的是 `-L` 的选项

```
$ sudo iptables -L -nv
Chain INPUT (policy ACCEPT 1368 packets, 8346K bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     udp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            udp dpt:53
    0     0 ACCEPT     tcp  --  virbr0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:53
```
iptables的每条规则都有两个计数器：
- `pkts`: 统计匹配到的报文的个数；
- `bytes`: 匹配到的所有报文的大小之和；

#### COMMAND 子命令格式
```
# rule-specification = [matches...]  [target]  
# match = -m matchname [per-match-options]  
# target = -j targetname [per-target-options]
iptables [-t table] {-A|-C|-D} chain   rule-specification  
iptables [-t table] -I chain [rulenum]   rule-specification  
iptables [-t table] -R chain rulenum   rule-specification  
iptables [-t table] -D chain rulenum  
iptables [-t table] -S [chain [rulenum]]  
iptables [-t table] {-F|-L|-Z} [chain [rulenum]]   [options...]  
iptables [-t table] -N chain  
iptables [-t table] -X  [chain]  
iptables [-t table] -P chain target  
iptables [-t table] -E old-chain-name   new-chain-name  
```

### 2.2 匹配条件：
`[-m matchname [per-match-options]]`
1. 基本匹配条件：无需加载任何模块，由iptables/netfilter自行提供；
2. 扩展匹配条件： 需要加载扩展模块，方可生效；

#### 基本匹配条件
- `[!] -s, --source  address[/mask][,...]`：
  - 作用: 检查报文中的源IP地址是否符合此处指定的地址或范围；
  - 附注: 所有地址可使用 `0.0.0.0/0` 或不指定次选项即可
- `[!] -d, --destination address[/mask][,...]`：
  - 作用: 检查报文中的目标IP地址是否符合此处指定的地址或范围；
  - 附注: 所有地址可使用 `0.0.0.0/0` 或不指定次选项即可
- `[!] -p, --protocol {tcp|udp|icmp}`：
  - 作用: 检查报文中的协议，即ip 首部中的 protocols 所标识的协议
  - protocol: tcp, udp, udplite, icmp, icmpv6,esp, ah, sctp, mh or  "all"
- `[!] -i, --in-interface IFACE`：
  - 作用: 数据报文流入的接口；只能应用于数据报文流入的环节，只能应用于PREROUTING，INPUT和FORWARD链；
- `[!] -o, --out-interface IFACE`：
  - 作用: 数据报文流出的接口；只能应用于数据报文流出的环节，只能应用于FORWARD、OUTPUT和POSTROUTING链                                       

#### 扩展匹配条件
扩展匹配条件可分为显示扩展和隐式扩展两种:
- 显式扩展：必须要手动加载扩展模块， `[-m matchname [per-match-options]]`；
- 隐式扩展：不需要手动加载扩展模块；因为它们是对协议的扩展，所以，但凡使用`-p`指明了协议，就表示已经指明了要扩展的模块；

常见协议的隐式扩展如下所示:

**tcp**
- `[!] --source-port, --sport port[-port]`：匹配报文的源端口；可以是端口范围；
- `[!] --destination-port,--dport port[-port]`：匹配报文的目标端口；可以是端口范围；
- `[!] --tcp-flags  list1  list2`
    - 作用: 检查list1 所指明的所有标志位，且这其中，list2 所列出的标志位必须为1，余下的必须为0，没在 list1 指明的，不做检查
    - 例如：`--tcp-flags  SYN,ACK,FIN,RST  SYN`表示，要检查的标志位为SYN,ACK,FIN,RST四个，其中SYN必须为1，余下的必须为0；
- `[!] --syn`：用于匹配第一次握手，相当于`--tcp-flags  SYN,ACK,FIN,RST  SYN`；
- 说明: 有关 tcp 连接的标识位，详见 [24.8 tcp 协议简述](24-iptables/tcp_protocol.md)

**udp**
- `[!] --source-port, --sport port[-port]`：匹配报文的源端口；可以是端口范围；
- `[!] --destination-port,--dport port[-port]`：匹配报文的目标端口；可以是端口范围；

**icmp**
- `[!] --icmp-type {type[/code]|typename}`
    - echo-request：ping 命令发送的请求的 icmp-type 为 8
    - echo-reply：ping 命令响应的 icmp-type 为 0

```
# 允许 ssh 连接
> iptables -t filter -A INPUT -d 192.168.1.105 -p tcp [-m tcp] --dport 22 -j ACCEPT
> iptables -t filter -A OUTPUT -s 192.168.1.105 -p tcp [-m tcp] --sport 22 -j ACCEPT

# 允许 ping 出，不允许 ping 入
> iptables -A OUTPUT -s 192.168.1.105 -p icmp --icmp-type 8 -j ACCEPT
> iptables -A INPUT -d 192.168.1.105 -p icmp --icmp-type 0 -j ACCEPT

# 通过规则设置默认策略
> iptables -P INPUT ACCEPT
> iptables -P OUTPUT ACCEPT
> iptables -F
> iptables -A INPUT  -d 192.168.1.168 -p tcp --dport 22 -j ACCEPT
> iptables -A OUTPUT  -s 192.168.1.168 -p tcp --sport 22 -j ACCEPT
> iptables -A OUTPUT -s 192.168.1.168 -p icmp --icmp-type 8 -j ACCEPT
> iptables -A INPUT -d 192.168.1.168 -p icmp --icmp-type 0 -j ACCEPT
> iptables -A INPUT -i enp0s3 -j DROP
> iptables -A OUTPUT -o enp0s3 -j DROP
> # iptables -L -nv
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  659 49584 ACCEPT     tcp  --  *      *       0.0.0.0/0            192.168.1.168        tcp dpt:22
    2   168 ACCEPT     icmp --  *      *       0.0.0.0/0            192.168.1.168        icmptype 0
   10   677 DROP       all  --  enp0s3 *       0.0.0.0/0            0.0.0.0/0           

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  311 30248 ACCEPT     tcp  --  *      *       192.168.1.168        0.0.0.0/0            tcp spt:22
    2   168 ACCEPT     icmp --  *      *       192.168.1.168        0.0.0.0/0            icmptype 8
   18  1304 DROP       all  --  *      enp0s3  0.0.0.0/0            0.0.0.0/0
```                                

## 2.3 处理动作：
`-j targetname [per-target-options]`
- `ACCEPT`: 接受
- `DROP`: 丢弃
- `REJECT`: 拒绝
- `RETURN`: 返回调用链；
- `REDIRECT`: 端口重定向；
- `LOG`: 记录日志；
- `MARK`: 做防火墙标记；
- `DNAT`: 目标地址转换；
- `SNAT`: 源地址转换；
- `MASQUERADE`: 地址伪装；
- 自定义链: 由自定义链上的规则进行匹配检查
- ......                    


### 课后作业：
开放本机web服务器给非192.168.0.0/24网络中的主机访问；
禁止本机被非172.16.0.0/16网络中的主机进行ping请求；
开放本机的dns服务给所有主机；
