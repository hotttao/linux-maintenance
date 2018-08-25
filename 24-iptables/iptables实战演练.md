# 24.4 iptables实战演练
## 2. 补充
利用iptables的recent模块来抵御DOS攻击:
1. 建立一个列表，保存有所有访问过指定的服务的客户端IP
2. ssh: 远程连接，

```
# one
> iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 3 -j DROP

# two
> iptables -I INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name SSH

# three
> iptables -I INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 300 --hitcount 3 --name SSH -j LOG --log-prefix "SSH Attach: "

# four
> iptables -I INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 300 --hitcount 3 --name SSH -j DROP

# 也可以使用下面的这句记录日志：
> iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --name SSH --second 300 --hitcount 3 -j LOG --log-prefix "SSH Attack"
```

1. one:
    - 利用connlimit模块将单IP的并发设置为3；会误杀使用NAT上网的用户，可以根据实际情况增大该值；
2. two:
    - 第二句是记录访问tcp 22端口的新连接，记录名称为SSH
    - --set 记录数据包的来源IP，如果IP已经存在将更新已经存在的条目
3. three
    - 利用recent和state模块限制单IP在300s内只能与本机建立2个新连接。被限制五分钟后即可恢复访问。
4. four:
    - 第三句是指SSH记录中的IP，300s内发起超过3次连接则拒绝此IP的连接。
    - --update 是指每次建立连接都更新列表；
    - --seconds必须与--rcheck或者--update同时使用
    - --hitcount必须与--rcheck或者--update同时使用
5. 附注:
    - iptables的记录：/proc/net/xt_recent/SSH


## 2. CentOS 6：
- http://ftp.redhat.com/redhat/linux/enterprise/6Server/en/os/SRPMS/
- layer7：第三方扩展；
    - iptables实现七层访问过滤：
    - 模块：layer7： 识别应用层协议
    - iptables/netfilter
        - iptables -m state,
        - netfilter state
- 编译:
    - 对内核中的netfilter，打补丁layer7，重新编译内核
    - 对iptables打补丁，补上layer7模块，重新iptables


## 3. diff/patch：文本操作工具
### 3.1 diff
- 简介: diff是Unix系统的一个很重要的工具程序。它用来比较两个文本文件的差异，是代码版本管理的核心工具之一。其用法非常简单：
- 命令: diff <变动前的文件> <变动后的文件>
- 版本: 由于历史原因，diff有三种格式：
    - 正常格式（normal diff）
    - 上下文格式（context diff）
    - 合并格式（unified diff）

````
# 正常格式的diff
例如，对file1（变动前的文件）和file2（变动后的文件）进行比较可使用如下命令：
> diff file1 file2

显示结果中，第一行是一个提示，用来说明变动位置。
它分成三个部分：
    前面的数字，表示file1的第n行有变化；
    中间的"c"表示变动的模式是内容改变（change），
    其他模式还有"增加"（a，代表addition）和"删除"（d，代表deletion）；

# 上下文格式的diff
上个世纪80年代初，加州大学伯克利分校推出BSD版本的Unix时，觉得diff的显示结果太简单，最好加入上下文，便于了解发生的变动。
因此，推出了上下文格式的diff。它的使用方法是加入-c选项（即context）
> diff -c f1 f2

结果分成四个部分
    第一部分的两行，显示两个文件的基本情况：文件名和时间信息，"***"表示变动前的文件，"---"表示变动后的文件。
    第二部分是15个星号，将文件的基本情况与变动内容分割开。
    第三部分显示变动前的文件，即file1,另外，文件内容的每一行最前面，还有一个标记位。
        如果为空，表示该行无变化；
        如果是感叹号（!），表示该行有改动；
        如果是减号（-），表示该行被删除；
        如果是加号（+），表示该行为新增。
    第四部分显示变动后的文件，即file2

# 合并格式的diff
如果两个文件相似度很高，那么上下文格式的diff，将显示大量重复的内容，很浪费空间。1990年，
GNU diff率先推出了"合并格式"的diff，将f1和f2的上下文合并在一起显示。它的使用方法是加入u参数（代表unified）
> diff -u f1 f2

结果:
    第一部分，也是文件的基本信息。"---"表示变动前的文件，"+++"表示变动后的文件。
    第二部分，变动的位置用两个@作为起首和结束。
    第三部分是变动的具体内容。除了有变动的那些行以外，也是上下文各显示3行。
        它将两个文件的上下文，合并显示在一起，所以叫做"合并格式"。
        每一行最前面的标志位，空表示无变动，减号表示第一个文件删除的行，加号表示第二个文件新增的行。
```


### 3.2 patch
- 简介:
    - 尽管并没有指定patch和diff的关系，但通常patch都使用diff的结果来完成打补丁的工作，
    - 这和patch本身支持多种diff输出文件格式有很大关系。patch通过读入patch命令文件（可以从标准输入），对目标文件进行修改。
    - 通常先用diff命令比较新老版本，patch命令文件则采用diff的输出文件，从而保持原版本与新版本一致。
- 命令: patch [options] [originalfile] [patchfile]
    - 如果patchfile为空则从标准输入读取patchfile内容；
    - 如果originalfile也为空，则从patchfile（肯定来自标准输入）中读取需要打补丁的文件名。
    - 因此，如果需要修改的是目录，一般都必须在patchfile中记录目录下的各个文件名。绝大多数情况下，patch都用以下这种简单的方式使用：
- 机制;
    - patch命令可以忽略文件中的冗余信息，从中取出diff的格式以及所需要patch的文件名，
    - 文件名按照diff参数中的"源文件"、"目标文件"以及冗余信息中的"Index："行中所指定的文件的顺序来决定。
- 参数:
    - -p参数: 决定了是否使用读出的源文件名的前缀目录信息，
    - 不提供-p参数，则忽略所有目录信息，
    - -p0（或者-p 0）表示使用全部的路径信息，
    - -p1将忽略第一个"/"以前的目录，
    - eg: 依此类推。如/usr/src/linux-2.4.15/Makefile这样的文件名，在提供-p3参数时将使用linux-2.4.15/Makefile作为所要patch的文件。


## 4. mockbuild
编译内核操作步骤
1. 获取并编译内核
2. 给内核打补丁
3. 编译并安装内核
4. 重启系统，启用新内核
5. 编译iptables
6. 为layer7模块提供其所识别的协议的特征码
7. 如何使用layer7模块
8. 提示：xt_layer7.ko依赖于nf_conntrack.ko模块

```
# 1. 获取并编译内核
> useradd mockbuild
> rpm -ivh kernel-2.6.32-431.5.1.x86_64.el6.src.rpm
> cd rpmbuild/SOURCES
> tar linux-2.6.32-*.tar.gz -C /usr/src
> cd /usr/src
> ln -sv

# 2. 给内核打补丁
> tar xf netfilter-layer7-v2.23.tar.bz2
> cd /usr/src/linux
> patch -p1 < /root/netfilter-layer7-v2.23/kernel-2.6.32-layer7-2.23.patch
> cp /boot/config-*  .config
> make menuconfig
# 2.1 按如下步骤启用layer7模块        
> Networking support → Networking Options →Network packet filtering framework → Core Netfilter Configuration
> <M>  “layer7” match support

# 3. 编译并安装内核
> make
> make modules_install
> make install

# 4. 重启系统，启用新内核

# 5. 编译iptables
> tar xf iptables-1.4.20.tar.gz
> cp /root/netfilter-layer7-v2.23/iptables-1.4.3forward-for-kernel-2.6.20forward/* /root/iptables-1.4.20/extensions/
> cp /etc/rc.d/init.d/iptales /root
> cp /etc/sysconfig/iptables-config /root
> rpm -e iptables iptables-ipv6 --nodeps
> ./configure  --prefix=/usr  --with-ksource=/usr/src/linux
> make && make install

> cp /root/iptables /etc/rc.d/init.d
> cp /root/iptables-config /etc/sysconfig

# 6. 为layer7模块提供其所识别的协议的特征码
> tar zxvf l7-protocols-2009-05-28.tar.gz
> cd l7-protocols-2009-05-28
> make install        

# 7. 如何使用layer7模块
# ACCT的功能已经可以在内核参数中按需启用或禁用。此参数需要装载nf_conntrack模块后方能生效。
> net.netfilter.nf_conntrack_acct = 1

# l7-filter uses the standard iptables extension syntax
> iptables [specify table & chain] -m layer7 --l7proto [protocol name] -j [action]
> iptables -A FORWARD -m layer7 --l7proto qq -j REJECT
```


### 编译内核：
```
make menuconfig
make -j #
make modules_install
make install
```
