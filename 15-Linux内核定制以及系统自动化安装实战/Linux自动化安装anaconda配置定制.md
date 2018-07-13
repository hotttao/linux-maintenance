# 15.2 Linux自动化安装anaconda配置定制
## 1. Centos 系统安装
### 1.1 安装程序：anaconda
anaconda：
- tui：基于cureses的文本配置窗口
- gui：图形界面


### 1.2 CentOS的安装过程启动流程：
1. Stage1：MBR：boot.cat
2. Stage2：isolinux/isolinux.bin
    - 配置文件：isolinux/isolinux.cfg
    - 每个对应的菜单选项：
        - 加载内核：isolinux/vmlinuz
        - 向内核传递参数：append  initrd=initrd.img .....
3. 装载根文件系统，并启动anaconda
    - 默认界面是图形界面：512MB+内存空间；
    - 若需要显式指定启动TUI接口： 向启动内核传递一个参数"text"即可；
        - 方法一： ESC, boot: linux  text
        - 方法二： table, text
4. 注意：上述内容一般位于引导设备，例如可通过光盘、U盘或网络等；后续的anacona及其安装用到的程序包等可以来自于程序包仓库，此仓库的位置可以为：
    - 本地光盘
    - 本地硬盘
    - ftp server
    - http server
    - nfs server
    - 如果想手动指定安装仓库：ESC,boot: linux method

### 1.2 anaconda的工作过程：
1. 安装前配置阶段
    - 安装过程使用的语言；
    - 键盘类型
    - 安装目标存储设备
        - Basic Storage：本地磁盘
        - Special Storage： iSCSI
    - 设定主机名
    - 配置网络接口
    - 时区
    - 管理员密码
    - 设定分区方式及MBR的安装位置；
    - 创建一个普通用户；
    - 选定要安装的程序包；
2. 安装阶段
    - 在目标磁盘创建分区并执行格式化；
    - 将选定的程序包安装至目标位置；
    - 安装bootloader；
3. 首次启动
    - iptables
    - selinux
    - core dump

### 1.3 anaconda的配置方式：
1. 交互式配置方式；
2. 支持通过读取配置文件中事先定义好的配置项自动完成配置；遵循特定的语法格式，此文件即为kickstart文件；

#### kickstart 文件格式
交互式配置安装完成后，在 root 目录下会生成此次安装的 kickstart 文件 - /root/anaconda-ks.cfg
1. 命令段：
    - 指定各种安装前配置选项，如键盘类型等；
        - 必备命令
        - 可选命令
2. 程序包段：
    - 指明要安装程序包，以及包组，也包括不安装的程序包；
        - %packages: 开始标记
        - @group_name: 包组
        - package: 单个包
        - -package: 不要安装的单个程序包
        - %end: 结束标记
3. 脚本段：
    - %pre：安装前脚本，运行环境：运行安装介质上的微型Linux系统环境；
    - %post：安装后脚本，运行环境：安装完成的系统；


**命令段中的必备命令**：
- authconfig：认证方式配置
    - authconfig  --enableshadow  --passalgo=sha512
- bootloader：定义bootloader的安装位置及相关配置
    - bootloader  --location=mbr  --driveorder=sda  --append="crashkernel=auto rhgb quiet"
- keyboard：设置键盘类型
    - keyboard us
- lang：语言类型
    - lang  zh_CN.UTF-8
- part：分区布局；
    - part  /boot  --fstype=ext4  --size=500
    - part  pv.008002  --size=51200
- rootpw：管理员密码
    - rootpw  --iscrypted  \$6\$4Yh15kMGDWOPtbbW$SGax4DsZwDAz4201.O97WvaqVJfHcISsSQEokZH054juNnoBmO/rmmA7H8ZsD08.fM.Z3Br/67Uffod1ZbE0s.
- timezone：时区
    - timezone  Asia/Shanghai
- 补充：分区相关的其它指令
    - clearpart：清除分区
        - clearpart  --none  --drives=sda：清空磁盘分区；
    - volgroup：创建卷组
        - volgroup  myvg  --pesize=4096  pv.008002
    - logvol：创建逻辑卷
        - logvol  /home  --fstype=ext4  --name=lv_home  --vgname=myvg  --size=5120
- 生成加密密码的方式：
    - openssl  passwd  -1  -salt `openssl rand -hex 4`

**可选命令**：
- install  OR  upgrade：安装或升级；
- text：安装界面类型，text为tui，默认为GUI
- network：配置网络接口
    - network  --onboot yes  --device eth0  --bootproto dhcp  --noipv6
- firewall：防火墙
    - firewall  --disabled
- selinux：SELinux
    - selinux --disabled
- halt、poweroff或reboot：安装完成之后的行为；
    - repo：指明安装时使用的repository；
        - repo  --name="CentOS"  --baseurl=cdrom:sr0  --cost=100
    - url： 指明安装时使用的repository，但为url格式；
        - url --url=http://172.16.0.1/cobbler/ks_mirror/CentOS-6.7-x86_64/    
- 参考官方文档：《Installation Guide》


#### 创建 kickstart 文件的方式
1. 直接手动编辑，或依据模板修改
2. 可使用创建工具 system-config-kickstart
    - 仅限 Centos 6
    - 也可依据模板修改并生成新配置
```
yum install  system-config-kickstart
system-config-kickstart

# 检查语法错误：
ksvalidator  /root/kickstart.cfg
```

### 1.4 安装引导选项：
boot:
1. text：文本安装方式
2. method：手动指定使用的安装方法
3. 与网络相关的引导选项：
    - ip=IPADDR
    - netmask=MASK
    - gateway=GW
    - dns=DNS_SERVER_IP
    - ifname=NAME:MAC_ADDR -- 指定上述设置应用在哪个网卡上
3. 远程访问功能相关的引导选项：
    - vnc
    - vncpassword='PASSWORD'
4. 启动紧急救援模式：
    - rescue
5. 装载额外驱动：
    - dd
6. 指定 kickstart 文件的位置
    - ks=
        - DVD drive: ks=cdrom:/PATH/TO/KICKSTART_FILE
        - Hard Drive： ks=hd:/DEVICE/PATH/TO/KICKSTART_FILE
        - HTTP Server： ks=http://HOST[:PORT]/PATH/TO/KICKSTART_FILE
        - FTP Server:  ks=ftp://HOST[:PORT]/PATH/TO/KICKSTART_FILE
        - HTTPS Server:  ks=https://HOST[:PORT]/PATH/TO/KICKSTART_FILE
7. 安装选项文档: www.redhat.com/docs , 《installation guide》        


### 1.5 创建引导光盘
```
> mkdir /tmp/myiso/isolinux
> cp /media/cdrom/isolinux/* /tmp/myiso/isolinux
> cp /root/kickstart.cfg /tmp/myiso/isoLinux
> mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS 6 x86_64 boot" -c isolinux/boot.cat -b isolinux/isolinux.bin -o  /root/boot.iso  myiso/

## 配置 isolinux/isolinux.cfg 添加安装项，直接配置 ks 参数
label linux ks
  menu
  menu
  kernal vmlinuz
  appeed initrd=initrd.img  ks=cdrom:/kickstart.cfg
```
