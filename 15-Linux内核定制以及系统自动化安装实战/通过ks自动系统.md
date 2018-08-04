# 15.4 通过 ks 自动安装系统

## 1. 通过 ks 利用光盘的额仓库安装操作系统
### 1.1 配置 http 服务器
 在 192.168.1.110 配置一个 http 服务器，让局域网内的所有机器都能访问到(http://192.168.1.110/anaconda-ks.cfg)

### 1.2 修改 kickstart 文件
`ksvalidator anaconda-ks.cfg`: 检查 ks 语法错误

```
#version=DEVEL
# System authorization information
auth --enableshadow --passalgo=sha512
# Use CDROM installation media
cdrom
# Use graphical install
graphical
# Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda
# Keyboard layouts
keyboard --vckeymap=cn --xlayouts='cn'
# System language
lang zh_CN.UTF-8

# Network information
network  --bootproto=dhcp --device=ens33 --onboot=off --ipv6=auto --no-activate
network  --hostname=www.tao.com

# Root password
rootpw --iscrypted $6$LZCrSYmUUKgH0NFI$T49uuvCjfCfl/7f87EZUHFBcqIRWjkhGeNuyHhGn/xUzv1o2sHefEH3AwHoMV7eVWY5rg2BarnuzlUkOCLgbL0
# System services
services --disabled="chronyd"
# System timezone
timezone Asia/Shanghai --isUtc --nontp
user --name=tao --password=$6$wpzzJIpol5z5KHw0$bsn4zRMv1hkBwg2HV8dqeE895i4YdgJU.J6q222HSec/sUBBPZflcdipfn9Z3U96mzlS48gZ5vFBAOG/WjV561 --iscrypted --gecos="tao"
# X Window System configuration information
xconfig  --startxonboot
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda

# Partition clearing information
clearpart --none --initlabel
part /boot  --fstype=ext4  --size=500
part  pv.008002  --size=20480
volgroup  cl  --pesize=4096  pv.008002
logvol  /  --fstype=xfs  --name=cl-root  --vgname=cl  --size=15360
logvol  /home  --fstype=xfs  --name=cl-home  --vgname=cl  --size=5120

%packages
@^developer-workstation-environment
@base
@core
@debugging
@desktop-debugging
@development
@dial-up
@directory-client
@fonts
@gnome-apps
@gnome-desktop
@guest-desktop-agents
@input-methods
@internet-applications
@internet-browser
@java-platform
@multimedia
@network-file-system-client
@performance
@perl-runtime
@print-client
@ruby-runtime
@virtualization-client
@virtualization-hypervisor
@virtualization-tools
@web-server
@x11
kexec-tools

%end

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
```

### 1.3 创建虚拟机并启动安装


## 2. 通过自制光盘使用网络仓库安装操作系统
### 2.1 创建磁盘映像文件
```
mount /dev/cdrom /media/cdrom/
cd /media/cdrom/
mkdir /root/myboot
cp -r isolinux/ /root/myboot/
cd /root/myboot/
cp /root/ks.cfg .

vim ks.cfg
# 更改 kickstart 文件添加
url --url=https://mirrors.aliyun.com/centos/7/os/x86_64/ 配置选项

vim isolinux/isolinux.cfg
# 更改菜单
label linux
  menu label ^Install tao linux
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet ks=cdrom:/ks.cfg


# 创建光盘镜像
cd /root
mkisofs -R -J -T -v --no-emul-boot --boot-load-size 4 --boot-info-table -V "CentOS 6 x86_64 boot" -c isolinux/boot.cat -b isolinux/isolinux.bin -o  /root/boot.iso   myboot/
```

### 1.2 修改 kickstart 文件


