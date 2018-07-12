# 14.6 Linux内核功能及模块应用
之前的章节中，我们讲解了 Linux 系统的启动流程，grub，以及 系统启动之后的 init 程序，最后我们来讲解 Linux 内核相关内容，包括
1. Linux 内核的组成，包括 内核，内核模块，ramdisk
2. 内核模块的管理
3. ramdisk 文件的制作
4. 内核参数的修改

## 1. Linux 内核设计体系
### 1.1 内核的组成部分：
Linux 是单内核设计，但引入了模块化机制，其组成包括如下几个部分
1. kernel：内核核心，一般为bzImage，通常位于/boot目录，名称为vmlinuz-VERSION-release；
2. kernel object：
    - 内核对象，即内核模块，一般放置于`/lib/modules/VERSION-release/` 
    - 内核模块与内核核心版本一定要严格匹配；
3. ramdisk：辅助性文件，并非必须，这取决于内核是否能直接驱动rootfs所在的设备。ramdisk 是一个简装版的根文件系统，可能包括
    - 目标设备驱动，例如SCSI设备的驱动；
    - 逻辑设备驱动，例如LVM设备的驱动；
    - 文件系统，例如xfs文件系统；
    

### 1.2 内核信息获取
#### uname命令：
`uname [OPTION]...`
- 作用: print system information
- 选项：
    - `-a`: 显示所有内核信息
    - `-r`：内核的release号
    - `-n`：主机名，节点名称

## 2. 模块信息获取和管理
### 2.1 模块信息获取
模块信息获取有 `lsmod`, `moinfo` 两个命令，它们的用法如下

#### lsmod
`lsmod`：
- 作用: 显示内核已经装载的模块
- 来源: 显示的内容来自于 `/proc/modules` 文件


#### modinfo
`modinfo [-F field] [-k kernel] [modulename|filename...]`：
- 作用: 查看单个模块的详细信息
- 选项: 
    - `-F field`： 仅显示指定字段的信息；
    - `-n`：显示模块文件路径；
    - `-p`：显示模块参数

```
> modinfo -F filename btrfs
> modinfo -n btrfs
```

#### depmod
`depmod`
- 作用: 内核模块依赖关系文件及系统信息映射文件的生成工具；

### 2.2 模块管理
模块装卸载有两组命令，一组是 `modprobe` 可以自动解决模块的依赖关系，另一组是 `insmod`,`rmmod` 不能自动解决模块的依赖关系，它们的用法如下


#### modprobe
`modprobe [ -C config-file] [-r]  module_name  [module params]`：
- 作用: 装载和卸载模块，会自动解决模块之间的依赖关系
- 选项:
    - 默认: 动态装载模块 `modprobe  module_name`
    - `-r`: 动态卸载     `modprobe  -r  module_name`
    - `-C`: 指定模块装载时的配置文件，默认为
        - `/etc/modprobe.conf`
        - `/etc/modprobe.d/\*.conf`   


#### insmod
`insmod  [filename]  [module options...]`：
- 作用: 模块的装载的另一命令，不会自动解决模块之间的依赖关系，不常用
- `filename`：模块文件的文件路径；
- eg: `insmod $(modinfo -n xfs)`

#### rmmod
`rmmod  [module_name]`：
- 作用: 模块卸载的另一命令
- `module_name`: 模块名称，不需要模块的路径



## 3. ramdisk文件的制作：
ramdisk 的制作有两个命令，Centos5 使用的 mkinitrd，Centos6-7 使用了 dracut，但是同时也提供了 mkinitrd, 其是基于 dracut 的脚本文件，它们的使用说明如下

#### mkinitrd
`mkinitrd [OPTION...] [<initrd-image>] <kernel-version>`
- 作用: 为当前使用中的内核重新制作ramdisk文件：
- 选项:
    - `--with=<module>`：除了默认的模块之外需要装载至initramfs中的模块；
    - `--preload=<module>`：initramfs所提供的模块需要预先装载的模块；
- eg： `mkinitrd  /boot/initramfs-$(uname -r).img  $(uname -r)`

#### dracut命令
`dracut [OPTION...] [<image> [<kernel version>]]`
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

## 4. 系统参数查看和修改
Linux 系统的所有参数通过 `/proc`，`/sys` 两个伪文件系统输出给用户查看和修改。

### 4.1 /proc 目录
/proc 的作用如下:
- 作用:
    - 内核把自己内核状态和统计信息，以及可配置参数通过 /proc 伪文件系统加以输出：
    - `/proc`：内核状态和统计信息的输出接口；
    - `/proc/sys`: 内核参数的配置接口 ；
- 内核参数分为
    - 只读：信息输出；例如`/proc/#/*`
    - 可写：可接受用户指定一个“新值”来实现对内核某功能或特性的配置；`/proc/sys/`

### 4.1 /proc/sys 管理工具
Linux 可修改的系统参数都放置在 /proc/sys 目录下，有三种修改方式
1. 通过 sysctl 命令，这是专用修改内核参数的命令
2. 由于 /proc 是伪文件系统，因此可以通过 cat，echo 等文件系统命令利用IO重定向进行修改，需要注意的是不能使用文本编辑器进行修改
3. 上述两种方式只临时有效，要想永久有效，需要修改配置文件

```
# 修改示例
> ls /proc/sys/kernel/hostname -l
> sysctl -w kernal.hostname='localhost'
> echo "localhost" > /proc/sys/kernel/hostname
```

#### sysctl命令
`sysctl [options]  [variable[=value]]`
- 作用: 专用于查看或设定/proc/sys目录下参数的值；
- 查看：
    - `sysctl  -a`: 查看所有参数
    - `sysctl  parameter`: 查看特定参数
- 修改：`sysctl  -w  parameter=value`
- 附注：sysctl 的内核参数是相对于 /proc/sys 目录下文件的相对路径而言的，比如 `/proc/sys/net/ipv4/ip_forward  相当于  net.ipv4.ip_forward`

```
# /proc/sys/net/ipv4/ip_forward
# parameter = net.ipv4.ip_forward
> sysctl -w net.ipv4.ip_forward=1
> echo 1 > /proc/sys/net/ipv4/ip_forward
```

#### 文件系统命令（cat, echo)
- 查看： `cat  /proc/sys/PATH/TO/SOME_KERNEL_FILE`
- 设定： `echo  "VALUE"  > /proc/sys/PATH/TO/SOME_KERNEL_FILE`


#### 修改配置文件
- 默认配置文件:
    - `/etc/sysctl.conf`
    - `/etc/sysctl.d/\*.conf`
- 配置文件立即生效：`sysctl  -p  [/PATH/TO/CONFIG_FILE]`


### 4.2 常用内核参数：
- **net.ipv4.ip_forward**：路由核心转发功能；
- **vm.drop_caches**：设置值为 1，将回收buffer，cache 的缓存
- **kernel.hostname**：主机名；
- **net.ipv4.icmp_echo_ignore_all**：忽略所有ping操作；

## 5. /sys目录：
/sys 目录目前主要的作用是
1. 输出内核识别出的各硬件设备的相关属性信息，
2. 也有内核对硬件特性的可设置参数；对此些参数的修改，即可定制硬件设备工作特性；

#### udev 命令
系统上所有的设备文件，都是由 udev 命令生成，这个命令的特点如下:
- udev 通过读取 /sys 目录下的硬件设备信息按需为各硬件设备创建设备文件；
- udev是用户空间程序；专用工具：devadmin, hotplug；
- udev为设备创建设备文件时，会读取其事先定义好的规则文件，一般在`/etc/udev/rules.d/`目录下，以及`/usr/lib/udev/rules.d/`目录下；

```
# 1. 修改 vmware 网卡的名称
> vim /usr/lib/udev/rules.d/70-persistent-net.rules
# 修改特定 mac 地址网卡的名称

# 2. 如果网卡有特定的配置信息，需要为网卡重新生成对应名称的配置文件
> cd /etc/sysconfig/net-work-script 


# 卸载网卡并重新装载网卡，让配置生效
> modprobe -r e1000
> modprobe e1000
```
