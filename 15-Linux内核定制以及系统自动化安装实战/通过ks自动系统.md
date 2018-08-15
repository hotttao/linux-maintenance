# 15.4 通过 ks 自动安装系统

## 1. 通过 ks 利用光盘的仓库安装操作系统
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
clearpart --initlabel --list=nvme0n1p11,nvme0n1p10,nvme0n1p8
# Disk partitioning information
part /boot/efi --fstype="efi" --ondisk=nvme0n1 --size=1028 --fsoptions="umask=0077,shortname=winnt"
part pv.1133 --fstype="lvmpv" --ondisk=nvme0n1 --size=153604
part /boot --fstype="xfs" --ondisk=nvme0n1 --size=1021
volgroup cl --pesize=4096 pv.1133
logvol /home  --fstype="xfs" --size=25600 --name=home --vgname=cl
logvol /var  --fstype="xfs" --size=46080 --name=var --vgname=cl
logvol swap  --fstype="swap" --size=2048 --name=swap --vgname=cl
logvol /  --fstype="xfs" --size=25600 --name=root --vgname=cl
logvol /usr  --fstype="xfs" --size=51200 --name=usr --vgname=cl

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
进入安装的 boot 界面输入
```
boot  linux  text ip=192.168.1.115 netmask=255.255.255.0 ks=http:/192.168.1.110/ks.cfg
```

## 2. 通过自制光盘自动安装操作系统
### 2.1 创建 kickstart 文件
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
network  --bootproto=dhcp --device=enp0s3 --onboot=off --ipv6=auto --no-activate
network  --hostname=virtual.tao

# Root password
rootpw --iscrypted $6$oSLEiL/Vx1k1thR7$5oER8NwNYgfdZcegPA6bBLyvMREZ5Pa6gEuikfDR.B09Mv7kWyJAaXAOoIbfBCZXDj91a5rBealE3S17i71.f1
# System services
services --enabled="chronyd"
# System timezone
timezone Asia/Shanghai --isUtc
user --groups=wheel --name=tao --password=$6$U7NwGZhelPqRaCP5$3VO3wcfzClT/nGXqoQobVN7.jlIfSTHDgUApHjAcwDhxROWK5/s3zZE0zUIaIhZse1OES30roxS1yxEQyydUv. --iscrypted --gecos="tao"
# X Window System configuration information
xconfig  --startxonboot
# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
# Partition clearing information
clearpart --none --initlabel
# Disk partitioning information
part pv.157 --fstype="lvmpv" --ondisk=sda --size=31747
part /boot --fstype="xfs" --ondisk=sda --size=1024
volgroup centos --pesize=4096 pv.157
logvol /  --fstype="xfs" --size=10240 --name=root --vgname=centos
logvol /var  --fstype="xfs" --size=10240 --name=var --vgname=centos
logvol /home  --fstype="xfs" --size=10240 --name=home --vgname=centos
logvol swap  --fstype="swap" --size=1020 --name=swap --vgname=centos

%packages
@^gnome-desktop-environment
@base
@core
@desktop-debugging
@development
@dial-up
@directory-client
@fonts
@gnome-desktop
@guest-agents
@guest-desktop-agents
@input-methods
@internet-browser
@java-platform
@multimedia
@network-file-system-client
@networkmanager-submodules
@print-client
@x11
chrony
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

### 2.2 创建磁盘映像文件
```bash
cd /var/iso
mkdir cdrom
mount -o loop CentOS-7-x86_64-DVD-1708.iso cdrom/
cp -ra cdrom/ myboot
vim myboot/ks.cfg   # 复制上述的 kickstart 文件


# 添加安装菜单
vim isolinux/isolinux.cfg
label ks
  menu label ^Install tao linux
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet ks=cdrom:/ks.cfg


# 创建光盘镜像
genisoimage -o CentOS-7.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -J -v -T  -V "CentOS 7 x86_64"  -eltorito-alt-boot    -bimages/efiboot.img      -no-emul-boot myboot/
# -V "CentOS 7 x86_64" 必需与 hd:LABEL=CentOS\x207\x20x86_64 保持一致
```


## 3. 通过自制光盘使用网络仓库安装操作系统
### 3.1 制作镜像文件
```
cd /var/iso
mount -o loop CentOS-7-x86_64-DVD-1708.iso cdrom/
cp -ra cdrom/isolinux/ myiso/
vim isolinux/isolinux.cfg # 添加开机菜单

label ks
  menu label Ks Install CentOS 7
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet ks=cdrom:/ks.cfg

cd myiso
cp /root/anaconda-ks.cfg ks.cfg

vim ks.cfg
  # Use CDROM installation media  更改为
  # cdrom
  url --url=https://mirrors.aliyun.com/centos/7/os/x86_64/

genisoimage -o Net.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -J -v -T  -V "CentOS 7 x86_64"  -eltorito-alt-boot    -bimages/efiboot.img      -no-emul-boot myiso
```

### 3.2 修改 kickstart 文件
ks 文件与上面的配置类似，但是需要使用 url 命令指定外部仓库的位置，将
```
# Use CDROM installation media  更改为
cdrom

# 更改为
url --url=https://mirrors.aliyun.com/centos/7/os/x86_64/  
```

### 3.3 创建虚拟机并启动安装
创建虚拟机后，选择配置的菜单，启动自动安装过程
