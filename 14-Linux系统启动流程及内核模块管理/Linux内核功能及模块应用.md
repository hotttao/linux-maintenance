# 14.6 Linux内核功能及模块应用

## 1. 内核设计体系：单内核、微内核
### 1.1 内核的组成部分：
1. Linux：单内核设计，但充分借鉴了微内核体系的设计的优点；为内核引入了模块化机制；
1. kernel：内核核心，一般为bzImage，通常位于/boot目录，名称为vmlinuz-VERSION-release；
2. kernel object：内核对象，即内核模块，一般放置于/lib/modules/VERSION-release/ 内核模块与内核核心版本一定要严格匹配；
3. ramdisk：辅助性文件，并非必须，这取决于内核是否能直接驱动rootfs所在的设备；
    - 目标设备驱动，例如SCSI设备的驱动；
    - 逻辑设备驱动，例如LVM设备的驱动；
    - 文件系统，例如xfs文件系统；
    - ramdisk：是一个简装版的根文件系统；

### 1.2 内核信息获取：
#### uname命令：
- 作用: print system information
- 选项：uname [OPTION]...
    - -a: 显示所有内核信息
    - -r：内核的release号
    - -n：主机名，节点名称

## 2. 模块信息获取和管理：
### 2.1 模块信息获取
**lsmod命令**：
- 作用: 显示内核已经装载的模块
- 来源: 显示的内容来自于 /proc/modules 文件

**modinfo命令**：
- 作用: 查看模块的详细信息
- 选项: modinfo [-F field] [-k kernel] [modulename|filename...]
    - -F field： 仅显示指定字段的信息；
    - -n：显示文件路径；
    - -p：显示模块参数

**depmod命令**：
- 作用: 内核模块依赖关系文件及系统信息映射文件的生成工具；

### 2.2 模块管理
**modprobe命令**：
- 作用: 装载和卸载模块，会自动解决模块之间的依赖关系
- 选项: modprobe [ -C config-file] [-r]  module_name  [module params]
    - 默认: 动态装载模块 modprobe  module_name
    - -r:  动态卸载    modprobe  -r  module_name
    - -C: 指定模块装载时的配置文件，默认为
        - /etc/modprobe.conf
        - /etc/modprobe.d/\*.conf    

**insmod命令**：
- 作用: 模块的装载的另一命令，不会自动解决模块之间的依赖关系，不常用
- 选项: insmod  [filename]  [module options...]
    - filename：模块文件的文件路径；
    - `insmod $(modinfo -n xfs)`

**rmmod命令**：
- 作用: 模块卸载的另一命令
- 选项: rmmod  [module_name]

## 3. ramdisk文件的制作：
#### mkinitrd命令
mkinitrd [OPTION...] [<initrd-image>] <kernel-version>
- 作用: 为当前使用中的内核重新制作ramdisk文件：
- 选项:
    - --with=<module>：除了默认的模块之外需要装载至initramfs中的模块；
    - --preload=<module>：initramfs所提供的模块需要预先装载的模块；
    - eg： `mkinitrd  /boot/initramfs-$(uname -r).img  $(uname -r)`

#### dracut命令
dracut [OPTION...] [<image> [<kernel version>]]
- 作用: low-level tool for generating an initramfs image
- eg： `dracut /boot/initramfs-$(uname -r).img    $(uname -r)`

#### 解开 ramdisk 文件
```
> mv initramfs-3.10.0-514.el7.x86_64.img initramfs-3.10.0-514.el7.x86_64.img.gz
> gzip -d initramfs-3.10.0-514.el7.x86_64.img.gz
> mkdir initrd
> cd initrd
> cpio -id < ../initramfs-3.10.0-514.el7.x86_64.img
```

## 4. /proc 目录
- 作用:
    - 内核把自己内核状态和统计信息，以及可配置参数通过 /proc 伪文件系统加以输出：
    - /proc：内核状态和统计信息的输出接口；
    - /proc/sys: 内核参数的配置接口 ；
- 内核参数
    - 只读：信息输出；例如/proc/#/*
    - 可写：可接受用户指定一个“新值”来实现对内核某功能或特性的配置；/proc/sys/

### 4.1 /proc/sys 管理工具
- net/ipv4/ip_forward  相当于  net.ipv4.ip_forward
- 参数配置:
    - sysctl 命令可查看和设定此目录中的参数
    - echo 命令通过重定向方式修改此目录中的参数
    - 注意：sysctl 和 echo 的设定仅当前运行内核有效；
```
# 修改示例
> ls /proc/sys/kernel/hostname -l
> sysctl -w kernal.hostname='localhost'
> echo "localhost" > /proc/sys/kernel/hostname
```

**sysctl命令**
sysctl [options]  [variable[=value]]
- 作用: 专用于查看或设定/proc/sys目录下参数的值；
- 查看：
    - sysctl  -a: 查看所有参数
    - sysctl  parameter
- 修改：sysctl  -w  parameter=value
```
# /proc/sys/net/ipv4/ip_forward
# parameter = net.ipv4.ip_forward
sysctl -w net.ipv4.ip_forward=1
echo 1 > /proc/sys/net/ipv4/ip_forward
```

**文件系统命令（cat, echo)**
- 查看： cat  /proc/sys/PATH/TO/SOME_KERNEL_FILE
- 设定： echo  "VALUE"  > /proc/sys/PATH/TO/SOME_KERNEL_FILE

**通过配置文件修改**：
- 默认配置文件:
    - /etc/sysctl.conf
    - /etc/sysctl.d/\*.conf
- 配置文件立即生效：sysctl  -p  [/PATH/TO/CONFIG_FILE]


### 4.2 常用内核参数：
- net.ipv4.ip_forward：路由核心转发功能；
- vm.drop_caches：设置值为 1，将回收buffer，cache 的缓存
- kernel.hostname：主机名；
- net.ipv4.icmp_echo_ignore_all：忽略所有ping操作；

## 5. /sys目录：
- sysfs：
    - 输出内核识别出的各硬件设备的相关属性信息，
    - 也有内核对硬件特性的可设置参数；对此些参数的修改，即可定制硬件设备工作特性；

**udev 命令**：
- 作用:
    - 通过读取/sys目录下的硬件设备信息按需为各硬件设备创建设备文件；
    - udev是用户空间程序；专用工具：devadmin, hotplug；
    - udev为设备创建设备文件时，会读取其事先定义好的规则文件，一般在/etc/udev/rules.d/目录下，以及/usr/lib/udev/rules.d/目录下；
