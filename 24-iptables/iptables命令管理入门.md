# 24.2 iptables命令管理入门

## 1. iptables/netfilter 过滤设置
#### 规则
- 组成：匹配条件 + 匹配成功后的处理动作；
- 匹配条件：根据协议报文特征指定
    - 基本匹配条件
    - 扩展匹配条件
- 处理动作：
    - 基本处理动作
    - 扩展处理动作
    - 自定义处理机制: 报文不会经过自定义链，只能在内置链上通过规则进行引用后生效
- 添加规则时的考量点：
    - 要实现哪种功能：判断添加到哪个表上；
    - 报文流经的路径：判断添加到哪个链上；

#### iptables的链：
内置链和自定义链
- 内置链：对应于hook function
- 自定义链接：用于内置链的扩展和补充，可实现更灵活的规则管理机制；

#### 链上的规则次序
即为检查的次序；因此，隐含一定的应用法则：
- 同类规则（访问同一应用），匹配范围小的放上面；
- 不同类的规则（访问不同应用），匹配到报文频率较大的放在上面；
- 将那些可由一条规则描述的多个规则合并起来；
- 设置默认策略；


## 2. iptables命令：
iptables [-t table] {-A|-C|-D} chain rule-specification
iptables [-t table] -I chain [rulenum] rule-specification
iptables [-t table] -R chain rulenum rule-specification
iptables [-t table] -D chain rulenum
iptables [-t table] -S [chain [rulenum]]
iptables [-t table] {-F|-L|-Z} [chain [rulenum]] [options...]
iptables [-t table] -N chain
iptables [-t table] -X  [chain]
iptables [-t table] -P chain target
iptables [-t table] -E old-chain-name new-chain-name
rule-specification = [matches...]  [target]
match = -m matchname [per-match-options]
target = -j targetname [per-target-options]

```
iptables  
1. [-t table]  COMMAND  chain          -- 基本命令          
2. [-m matchname [per-match-options]]  -- 匹配条件  
3. -j targetname [per-target-options]  -- 处理动作  
```

### 2.1 基本命令    
iptables  [-t table]  COMMAND  chain
- -t table：raw, mangle, nat, [filter]
- COMMAND：
    - 链管理：
        - -F：flush，清空指定的规则链；省略表示清空指定表上的所有链
        - -N：new, 自定义一条新的规则链；
        - -X：delete，删除自定义的空的规则链；
        - -Z：zero，将规则计数器置零；
        - -P：Policy，为指定链设置默认策略；对filter表中的链而言，其默认策略有：
            - ACCEPT：接受
            - DROP：丢弃
            - REJECT：拒绝
        - -E：重命名自定义链；引用计数不为0的自定义链不能够被重命名，也不能被删除；
    - 规则管理：
        - -A：append，追加；
        - -I：insert, 插入，要指明位置，省略时表示第一条；
        - -D：delete，删除；
            - 指明规则序号；
            - 指明匹配条件；
        - -R：replace，替换指定链上的指定规则；
    - 附注:
        - iptables的每条规则都有两个计数器：
            - pkts:匹配到的报文的个数；
            - bytes:匹配到的所有报文的大小之和；                        
    - 查看：
        - -L：list, 列出指定鏈上的所有规则；
        - -n：numberic，以数字格式显示地址和端口号(不对ip地址进行反解)；
        - -v：verbose，详细信息；
        - -vv, -vvv
        - -x：exactly，显示计数器结果的精确值；
        - --line-numbers：显示规则的序号；
- chain: PREROUTING，INPUT，FORWARD，OUTPUT，POSTROUTING

### 2.2 匹配条件：
`[-m matchname [per-match-options]]`  -- 指定扩展模块，用于扩展匹配
1. 基本匹配条件：无需加载任何模块，由iptables/netfilter自行提供；
2. 扩展匹配条件： 需要加载扩展模块，方可生效；

基本匹配条件
- `[!] -s, --source  address[/mask][,...]`：检查报文中的源IP地址是否符合此处指定的地址或范围；
- `[!] -d, --destination address[/mask][,...]`：检查报文中的目标IP地址是否符合此处指定的地址或范围；
- `[!] -p, --protocol {tcp|udp|icmp}`：检查报文中的协议，即ip 首部中的 protocols 所标识的协议
    - protocol: tcp, udp, udplite, icmp, icmpv6,esp, ah, sctp, mh or  "all"
- `[!] -i, --in-interface IFACE`：数据报文流入的接口；只能应用于数据报文流入的环节，只能应用于PREROUTING，INPUT和FORWARD链；
- `[!] -o, --out-interface IFACE`：数据报文流出的接口；只能应用于数据报文流出的环节，只能应用于FORWARD、OUTPUT和POSTROUTING链；                                            

扩展匹配条件
- 显式扩展：必须要手动加载扩展模块， [-m matchname [per-match-options]]；
- 隐式扩展：不需要手动加载扩展模块；因为它们是对协议的扩展，所以，但凡使用-p指明了协议，就表示已经指明了要扩展的模块；
- tcp：
    - [!] --source-port, --sport port[-port]：匹配报文的源端口；可以是端口范围；
    - [!] --destination-port,--dport port[-port]：匹配报文的目标端口；可以是端口范围；
    - [!] --tcp-flags  list1  list2
        - 作用: 检查list1 所指明的所有标志位，且这其中，list2 所列出的标志位必须为1，余下的必须为0，没在 list1 指明的，不做检查
        - 例如：“--tcp-flags  SYN,ACK,FIN,RST  SYN”表示，要检查的标志位为SYN,ACK,FIN,RST四个，其中SYN必须为1，余下的必须为0；
    - [!] --syn：用于匹配第一次握手，相当于”--tcp-flags  SYN,ACK,FIN,RST  SYN“；                                
- udp
    - [!] --source-port, --sport port[-port]：匹配报文的源端口；可以是端口范围；
    - [!] --destination-port,--dport port[-port]：匹配报文的目标端口；可以是端口范围；
- icmp
    - [!] --icmp-type {type[/code]|typename}
        - echo-request：8
        - echo-reply：0
```
# 允许 ssh 连接
> iptables -t filter -A INPUT -d 192.168.1.105 -p tcp [-m tcp] --dport 22 -j ACCEPT
> iptables -t filter -A OUTPUT -s 192.168.1.105 -p tcp [-m tcp] --sport 22 -j ACCEPT

# 允许 ping 出，不允许 ping 入
> iptables -A OUTPUT -s 192.168.1.105 -p icmp --icmp-type 8 -j ACCEPT
> iptables -A INPUT -d 192.168.1.105 -p icmp --icmp-type 0 -j ACCEPT
```                                

## 2.3 处理动作：
`-j targetname [per-target-options]`
- ACCEPT: 接受
- DROP: 丢弃
- REJECT: 拒绝
- RETURN：返回调用链；
- REDIRECT：端口重定向；
- LOG：记录日志；
- MARK：做防火墙标记；
- DNAT：目标地址转换；
- SNAT：源地址转换；
- MASQUERADE：地址伪装；
- ...
- 自定义链：由自定义链上的规则进行匹配检查


### 3. 防火墙（服务）：
1. CentOS 6：
    - service  iptables  {start|stop|restart|status}
        - start：读取事先保存的规则，并应用到netfilter上；
        - stop：清空netfilter上的规则，以及还原默认策略等；
        - status：显示生效的规则；
        - restart：清空netfilter上的规则，再读取事先保存的规则，并应用到netfilter上；
    - 默认的规则文件：/etc/sysconfig/iptables
2. CentOS 7：
    - systemctl  start|stop|restart|status  firewalld.service
    - systemctl  disable  firewalld.service
    - systemctl  stop  firewalld.service                    


### 课后作业：
开放本机web服务器给非192.168.0.0/24网络中的主机访问；
禁止本机被非172.16.0.0/16网络中的主机进行ping请求；
开放本机的dns服务给所有主机；
