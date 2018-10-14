# 44.3 kvm 管理工具 qemu-kvm

## 3. 使用 qemu-kvm 创建虚拟机
### 3.1 创建虚拟机示例
```
# cirros project: 为cloud环境测试vm提供的微缩版Linux；
# 1. 启动第一个虚拟：
qemu-kvm -m 128 -smp 2 -name test -hda /images/kvm/cirros-0.3.4-i386.disk.img

# 2. 用-drive指定磁盘映像文件：
qemu-kvm -m 128 -name test -smp 2 -drive file=/images/kvm/cirros-0.3.4-i386-disk.img, \
                if=virtio,media=disk,cache=writeback,format=qcow2

# 3. 通过cdrom启动winxp的安装：
qemu-kvm -name winxp -smp 4,sockets=1,cores=2,threads=2 -m 512 -drive  \
                  file=/images/kvm/winxp.img,if=ide,media=disk,
                  cache=writeback,format=qcow2 -drive file=/root/winxp_ghost.iso,media=cdrom

# 4. 指定使用桥接网络接口：
qemu-kvm -m 128 -name test -smp 2 -drive file=/images/kvm/cirros-0.3.4-i386-disk.img, \
                 if=virtio,media=disk,cache=writeback,format=qcow2 -net nic -net tap, \
                 script=/etc/if-up,downscript=no -nographic

示例1：
	 ~]#  qemu-kvm -name c2 -smp 2,maxcpus=4,sockets=2,cores=2 -m 128 -drive file=/images/kvm/cos-i386.qcow2,if=virtio -vnc  :1 -daemonize -net nic,model=e1000,macaddr=52:54:00:00:00:11 -net tap,script=/etc/qemu-ifup
示例2：	 
	 ~]# qemu-kvm -name winxp -smp 1,maxcpus=2,sockets=1,cores=2 -m 1024 -drive file=/data/vms/winxp.qcow2,media=disk,cache=writeback,format=qcow2 file=/tmp/winxp.iso,media=cdrom -boot order=dc,once=d -vnc :1 -net nic,model=rtl8139,macaddr=52:54:00:00:aa:11 -net tap,ifname=tap1,script=/etc/qemu-ifup -daemonize
```

### 2.2  创建成功的反馈信息
```
# 显示选项：
# 1. SDL: Simple DirectMedia Layer：C语言开发，跨平台且开源多媒体程序库文件；在qemu中使用“-sdl”即可；
# 2. VNC: Virtual Network Computing，使用RFB(Remote FrameBuffer)协议远程控制另外的主机；
# 安装使用 vnc
# CentOS 6.6
yum install tigervnc-server
vncpasswd
vncserver :N

# Centos 7
yum install tigervnc
vncviewer  :N
# qemu-kvm -vnc display,option,option
# 示例：-vnc :N,password
# 启动qemu-kvm时，额外使用-monitor stdio选项，并使用
# change vnc password命令设置密码；
```      

## 3. qemu-kum使用文档
`qemu-kvm [options] [disk_image]`
options:
- 标准选项；
- USB选项；
- 显示选项；
- i386平台专用选项；
- 网络选项；
- 字符设备选项；
- 蓝牙相关选项；
- Linux系统引导专用选项；
- 调试/专家模式选项；
- PowerPC专用选项；
- Sparc32专用选项；

### 3.1 标准选项
作用: 主要涉及指定主机类型、CPU模式、NUMA、软驱设备、光驱设备及硬件设备等。
选项:
- `-name name`：设定虚拟机名称；
- `-M machine`：
	- 指定要模拟的主机类型，如Standard PC、ISA-only PC或Intel-Mac等，
	- 可以使用`qemu-kvm -M ?`获取所支持的所有类型；
- `-m megs`：设定虚拟机的RAM大小；
- `-cpu model`：
	- 设定CPU模型，如coreduo、qemu64等，
	- 可以使用`qemu-kvm -cpu ?`获取所支持的所有模型；
- `-smp n[,cores=cores][,threads=threads][,sockets=sockets][,maxcpus=maxcpus]`：
      	- 参数:
      		- n: 几颗CPU
      		- sockets: 一颗CPU上的卡槽数
      		- cores: CPU 的核心数
      		- threads: 一个核心的线程数
      	- 设定模拟的SMP架构中CPU的个数等、每个CPU的核心数及CPU的socket数目等；
      	- PC机上最多可以模拟255颗CPU；
      	- maxcpus用于指定热插入的CPU个数上限；
- `-numa opts`：指定模拟多节点的numa设备；
- `-fda file`
- `-fdb file`：使用指定文件(file)作为软盘镜像，file为/dev/fd0表示使用物理软驱；
- `-hda file`
- `-hdb file`
- `-hdc file`
- `-hdd file`：使用指定file作为硬盘镜像；
- `-cdrom file`：
	- 使用指定file作为CD-ROM镜像
	- 需要注意的是-cdrom和-hdc不能同时使用
	- 将file指定为/dev/cdrom可以直接使用物理光驱；
- `-drive option[,option[,option[,...]]]`：定义一个硬盘设备；可用子选项有很多。
	- `file=/path/to/somefile`：硬件映像文件路径；
	- `if=interface`：指定硬盘设备所连接的接口类型，即控制器类型，如ide、scsi、sd、mtd、floppy、pflash及virtio等；
	- `index=index`：设定同一种控制器类型中不同设备的索引号，即标识号；
	- `media=media`：定义介质类型为硬盘(disk)还是光盘(cdrom)；
	- `snapshot=snapshot`：指定当前硬盘设备是否支持快照功能：on或off；
	- `cache=cache`：定义如何使用物理机缓存来访问块数据，其可用值有none、writeback、unsafe和writethrough四个；
	- `format=format`：指定映像文件的格式，具体格式可参见qemu-img命令；
- `-boot [order=drives][,once=drives][,menu=on|off]`：
	- 定义启动设备的引导次序，每种设备使用一个字符表示；
	- 不同的架构所支持的设备及其表示字符不尽相同
	- 在x86 PC架构上，a、b表示软驱、c表示第一块硬盘，d表示第一个光驱设备，n-p表示网络适配器；默认为硬盘设备；
	- eg: `-boot order=dc,once=d` -  第一次开机默认以光盘作为首选开启启动项目
### 3.2 qemu-kvm的显示选项
作用: 显示选项用于定义虚拟机启动后的显示接口相关类型及属性等。
```
-display sdl[,frame=on|off][,alt_grab=on|off][,ctrl_grab=on|off]
            [,window_close=on|off]|curses|none|
            vnc=<display>[,<optargs>]
```
- `-nographic`：
	- 默认情况下，qemu使用SDL来显示VGA输出；
	- 而此选项用于禁止图形接口，此时,qemu类似一个简单的命令行程序，其仿真串口设备将被重定向到控制台；
- `-curses`：- 禁止图形接口，并使用curses/ncurses作为交互接口；
- `-alt-grab`：使用Ctrl+Alt+Shift组合键释放鼠标；
- `-ctrl-grab`：使用右Ctrl键释放鼠标；
- `-sdl`：启用SDL；
- `-spice option[,option[,...]]`：启用spice远程桌面协议；其有许多子选项，具体请参照qemu-kvm的手册；
- `-vga type`：指定要仿真的VGA接口类型，常见类型有：
	- `cirrus`：Cirrus Logic GD5446显示卡；
	- `std`：带有Bochs VBI扩展的标准VGA显示卡；
	- `vmware`：VMWare SVGA-II兼容的显示适配器；
	- `qxl`：QXL半虚拟化显示卡；与VGA兼容；在Guest中安装qxl驱动后能以很好的方式工作，在使用spice协议时推荐使用此类型；
	- `none`：禁用VGA卡；
- -`vnc display[,option[,option[,...]]]`：
	- 默认情况下，qemu使用SDL显示VGA输出；
	- 使用-vnc选项，可以让qemu监听在VNC上，并将VGA输出重定向至VNC会话；
	- 使用此选项时，必须使用-k选项指定键盘布局类型；
	- 其有许多子选项，具体请参照qemu-kvm的手册；

```
display:
(1) host:N
	172.16.100.7:1, 监听于172.16.100.7主的5900+N的端口上
(2) unix:/path/to/socket_file
(3) none
options:
	password: 连接时需要验正密码；设定密码通过monitor接口使用change
	reverse: “反向”连接至某处于监听状态的vncview上；
-monitor stdio：表示在标准输入输出上显示monitor界面
-nographic
	Ctrl-a, c: 在console和monitor之间切换
	Ctrl-a, h: 显示帮助信息
```

### 3.3 386平台专用选项
- `-no-acpi`：禁用ACPI功能，GuestOS与ACPI出现兼容问题时使用此选项；
- `-balloon none`：禁用balloon设备；
- `-balloon virtio[,addr=addr]`：启用virtio balloon设备；
### 3.4 网络属性相关选项
作用: 用于定义网络设备接口类型及其相关的各属性等信息。这里只介绍nic、tap和user三种类型网络接口的属性，其它类型请参照qemu-kvm手册。
选项:
1. `-net nic[,vlan=n][,macaddr=mac][,model=type][,name=name][,addr=addr][,vectors=v]`：
	- 创建一个新的网卡设备并连接至vlan n中；
	- PC架构上默认的NIC为e1000
	- macaddr用于为其指定MAC地址
	- name用于指定一个在监控时显示的网上设备名称
	- emu可以模拟多个类型的网卡设备，
		- 如virtio、i82551、i82557b、i82559er、ne2k_isa、pcnet、rtl8139、e1000、smc91c111、lance及mcf_fec等；
		- 不同平台架构上，其支持的类型可能只包含前述列表的一部分
		- 可以使用`qemu-kvm -net nic,model=?`来获取当前平台支持的类型；
2. `-net tap[,vlan=n][,name=name][,fd=h][,ifname=name][,script=file][,downscript=dfile]`：
	- 通过物理机的TAP网络接口连接至vlan n中，
	- 使用script=file指定的脚本(默认为/etc/qemu-ifup)来配置当前网络接口，
	- 使用downscript=file指定的脚本(默认为/etc/qemu-ifdown)来撤消接口配置；
	- 使用script=no和downscript=no可分别用来禁止执行脚本；
3. `-net user[,option][,option][,...]`：在用户模式配置网络栈，其不依赖于管理权限；有效选项有：
	- `vlan=n`：连接至vlan n，默认n=0；
	- `name=name`：指定接口的显示名称，常用于监控模式中；
	- `net=addr[/mask]`：设定GuestOS可见的IP网络，掩码可选，默认为10.0.2.0/8；
	- `host=addr`：指定GuestOS中看到的物理机的IP地址，默认为指定网络中的第二个，即x.x.x.2；
	- `dhcpstart=addr`：指定DHCP服务地址池中16个地址的起始IP，默认为第16个至第31个，即x.x.x.16-x.x.x.31；
	- `dns=addr`：指定GuestOS可见的dns服务器地址；默认为GuestOS网络中的第三个地址，即x.x.x.3；
	- `tftp=dir`：激活内置的tftp服务器，并使用指定的dir作为tftp服务器的默认根目录；
	- `bootfile=file`：
		- BOOTP文件名称，用于实现网络引导GuestOS；
		- 如：`qemu -hda linux.img -boot n -net user,tftp=/tftpserver/pub,bootfile=/pxelinux.0`
```
# cat /etc/qemu-ifup
#!/bin/bash
#
bridge=br0
if [ -n "$1" ]; then
	ip link set $1 up
	sleep 1
	brctl addif $bridge $1
[ $? -eq 0 ] && exit 0 || exit 1
	else
	echo "Error: no interface specified."
exit 1
fi
```

```
# cat /etc/qemu-ifdown
#!/bin/bash
#
bridge=br0
if [ -n "$1" ];then
	brctl delif $bridge $1
	ip link set $1 down
	exit 0
else
	echo "Error: no interface specified."
	exit 1
fi
```

### 3.5 一个使用示例
```
# 下面的命令创建了一个名为rhel5.8的虚拟机，
# 其RAM大小为512MB，有两颗CPU的SMP架构，默认引导设备为硬盘，
# 有一个硬盘设备和一个光驱设备，网络接口类型为virtio，VGA模式为cirrus，并启用了balloon功能。
# 需要注意的是，命令中使用的硬盘映像文件/VM/images/rhel5.8/hda需要事先使用qemu-img命令创建
qemu-kvm -name "rhel5.8" -m 512 \
		-smp 2 -boot d \
		-drive file=/VM/images/rhel5.8/hda,if=virtio,index=0,media=disk,format=qcow2 \
		-drive file=/isos/rhel-5.8.iso,index=1,media=cdrom \
		-net nic,model=virtio,macaddr=52:54:00:A5:41:1E \
		-vga cirrus -balloon virtio

# 在虚拟机创建并安装GuestOS完成之后，可以免去光驱设备直接启动之。命令如下所示。
qemu-kvm -name "rhel5.8" -m 512 \
		-smp 2 -boot d \
		-drive file=/VM/images/rhel5.8/hda,if=virtio,index=0,media=disk,format=qcow2 \
		-net nic,model=virtio,macaddr=52:54:00:A5:41:1E \
		-vga cirrus -balloon virtio
```

## 4. 使用qemu-img管理磁盘映像
`qemu-img  subcommand  [options]`
1. 作用:  qemu-img是qemu用来实现磁盘映像管理的工具组件，其有许多子命令，分别用于实现不同的管理功能，
2. subcommand
	- `create`：创建一个新的磁盘映像文件；
	- `check`：检查磁盘映像文件中的错误；
	- `convert`：转换磁盘映像的格式；
	- `info`：显示指定磁盘映像的信息；
	- `snapshot`：管理磁盘映像的快照；
	- `commit`：提交磁盘映像的所有改变；
	- `rbase`：基于某磁盘映像创建新的映像文件；
	- `resize`：增大或缩减磁盘映像文件的大小；

### 4.1 create子命令
`create [-f fmt] [-o options] filename [size]`

```
# 例如下面的命令创建了一个格式为qcow2的120G的稀疏磁盘映像文件。
qemu-img create -f qcow2  /VM/images/rhel5.8/hda 120G
# Formatting '/VM/images/rhel5.8/hda', fmt=qcow2 size=128849018880 encryption=off cluster_size=65536
```
