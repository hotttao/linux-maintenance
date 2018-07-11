# 14.1 CentOS系统启动流程介绍
本节，我们开始学习 Centos 系统的启动流程，本章开篇我们了解到，开机过程中存在两个问题:
1. 为加载内核需要读取磁盘中的内核源码，而读取磁盘文件需要先挂载根文件系统，此时操作系统还没有更不可能挂在根文件系统了
2. 挂载根文件系统时，首先需要加载磁盘和文件系统的驱动程序。而文件系统还没有挂载，根本没法读取到位于根文件系统中的驱动程序

本节核心就是讲解如何解决上述两个问题，并在此基础上介绍 Linux 系统的启动。本节将包括以下内容:
1. Linux 系统特性，包括Linux 系统的功能及设计思路
2. Linux 开机启动流程，包括:
	- Linux 系统组成，Linux 为开机启动准备的额外文件
	- Linux 系统详细启动过程

## 1. Linux 系统特性
Linux 主要由内核+根文件系统组成，一个运行中的Linux，可以看作是运行在内核之上，由内核负责完成底层硬件管理，从而将硬件接口抽象为系统接口以后，让根文件系统工作为文件系统的一个底层虚拟机。运行中的OS可以分为内核空间和用户空间两个部分，应用程序运行在用户空间，通常表现为一个进程或线程；而内核空间主要运行内核代码，执行特权操作，通过系统调用向用户空间输出接口。用户空间通过发起系统调用执行特权操作。

Linux 需要实现的功能包括:
1. 进程管理，进程创建，调度，销毁
2. 内存管理，将内存抽象为虚拟的线性地址格式，为每个进程提供一份，就好像每个进程单独运行在操作系统之上一样
3. IPC 机制:
	- 消息队列
	- semerphor 信号量
	- share memory 共享内存
4. 网络协议栈
5. 文件系统
6. 驱动程序 《Linux 设备驱动》
7. 安全功能

内核设计有两种流派
1. 单内核设计：把所有功能集成于同一个程序，典型代表为 Linux
2. 微内核设计：每种功能使用一个单独的子系统实现，典型代表为 Windows, Solaris

Linux 虽然为单内核设计但是充分吸收了微内核的特点，支持模块化(.ko (kernel object))，支持模块运行时动态装载或卸载。

## 1. Linux 系统启动流程
### 1.1 Linux 系统的组成
首先我们来解决开机启动的第二个问题。
内核加载根文件系统需要加载驱动程序，而驱动程序就在根之上。因此我们不能依赖根文件系统上的驱动程序，内核必须自带驱动程序。

一种方法是将设备的驱动程序编译进内核，对于个人用户自编译的系统没有有问题，因为只需要将其特定的驱动程序编译进内核即可，然后对于操作系统发行商而然，它面对的是各种用户的不同驱动设备，如果都将这些驱动程序编译进内核，内核将庞大无比，而每个用户又只会用到其中一个。

另一种方法是借助于中间临时根文件系统，中间临时根文件系统包含了加载根文件系统所在设备的驱动程序，而中间根文件系统放置在一个基于内存的磁盘设备中，内核无须加载其他驱动程序即可访问该设备。内核启动后，先访问基本设备挂载中间的临时根文件系统，并从中装载设备驱动程序，在真正的根文件系统准备完成之后，从临时根切换到真正的根。

用于系统初始化的基于内存的磁盘设备通常称为 ramdisk，内核在启动过程中需要将 ranmdisk 装载进内存 ，并将其识别为一个根文件系统。ramdisk 并不是发行商预先生成，而是在系统安装过程中针对当前设备临时生成了，因此其仅需包含当前设备的驱动程序即可。

因此从编译完成后的视角，Linux 系统由如下部分组成:
1. 核心文件：/boot/vmlinuz-VERSION-release
2. ramdisk：
	- CentOS 5：`/boot/initrd-VERSION-release.img`   # 基于 ram 的磁盘
	- CentOS 6,7：`/boot/initramfs-VERSION-release.img`  # 基于 ram 的文件系统
3. 模块文件：
	- `/lib/modules/VERSION-release`
	- 如果内核提供了多个版本，将会有多个内核目录

```
> uname -r # 内核版本
4.9.86-30.el7.x86_64

> ls /boot/|grep vm
vmlinuz-4.9.86-30.el7.x86_64

> ls /lib/modules/$(uname -r)/kernel
arch  crypto  drivers  fs  lib  mm  net  sound  virt

> tree  -L 2 /lib/modules/$(uname -r)
/lib/modules/4.9.86-30.el7.x86_64
├── build -> ../../../usr/src/kernels/4.9.86-30.el7.x86_64
├── extra
├── kernel
│   ├── arch
│   ├── crypto
│   ├── drivers
│   ├── fs
│   ├── lib
│   ├── mm
│   ├── net
│   ├── sound
│   └── virt
├── modules.alias
├── modules.alias.bin
├── modules.block
├── modules.builtin
├── modules.builtin.bin
├── modules.dep
├── modules.dep.bin
├── modules.devname
├── modules.drm
├── modules.modesetting
├── modules.networking
├── modules.order
├── modules.softdep
├── modules.symbols
├── modules.symbols.bin
├── source -> build
├── updates
├── vdso
│   ├── vdso32.so
│   └── vdso64.so
└── weak-updates
```

### 1.2 MBR 与 BootSector
接下来我们来解决第一个问题，在没有根文件系统的前提下将内核加载进内存。
可引导设备的第一个分区叫MBR，MBR 中包含了开机引导程序 BootLoader。开机启动时会先加载 MBR内的BootLoader，由BootLoader 将内核加载到内存。有人可能会问，开机时是如何读取到MBR的，BootLoader又是如何读取到内核文件的。BIOS 通过硬件的 INT 13 中断功能来读取 MBR，也就是说，只要 BIOS 能够侦测的到你的磁盘 （不论磁盘是 SATA 还是 SAS ），那他就有办法通过 INT 13 这条信道来读取该磁盘的第一个扇区内的 MBR 软件，这样 boot loader 也就能够被执行。boot loader 能够识别操作系统的文件格式，也就能加载核心文件。其他分区的第一个扇区叫做 boot sector，也可以安装BootLoader，这样可以实现多系统安装。

有了上述阐述，我们就可以开始讲解开机启动流程了。

### 1.2 Centos 系统的启动流程(MBR 架构)
启动流程:
1. POST: 加电自检。
	- x86 架构的计算机被设计成，只要通电就会去执行，主板上有个 ROM 芯片内的BOIS程序
	- 通过 BIOS 程序去载入 CMOS 的信息，并且借由 CMOS 内的设置值取得主机的各项硬件参数及设置，例如硬盘的大小与类型
	- 在获取硬件信息后，BIOS 会进行开机自我测试 (Power-on Self Test, POST) ，然后开始执行硬件侦测的初始化，并设置 PnP 设 备，之后再定义出可开机的设备顺序，接下来就会开始进行开机设备的数据读取
2. Boot Sequence:开机启动次序
	- 家电自检完成后，计算机就会按次序查找各引导设备，第一个有引导程序的设备即为本次启动用到的设备。
	- 引导程序称为 BootLoader，又称引导加载器。
	- 如果是通过U盘安装操作系统，就需要进入 BIOS 设置系统的开机启动次序
3. bootloader：引导加载器，程序，位于MBR中；
    - 功能：
        - 提供一个菜单，允许用户选择要启动的系统或不同的内核版本；
        - 把用户选定的内核装载到RAM中的特定空间中，解压、展开，而后把系统控制权移交给内核
    - Windows：ntloader
    - Linux：
        - LILO：LIinux  LOader
        - GRUB：Grand Uniform Bootloader
            - GRUB 0.X：Grub Legacy
            - GRUB 1.X：Grub2
    - MBR/GRUB:
        - MBR：Master Boot Record，512bytes：
            - 446bytes：bootloader
            - 64bytes：fat, 磁盘分区表
            - 2bytes：55AA                    
        - GRUB：两阶段加载
            - bootloader：1st stage
            - Partition：filesystem driver, 1.5 stage
            - Partition：/boot/grub, 2nd stage
4. Kernel:
	- 自身初始化：
        - 探测可识别到的所有硬件设备；
        - 加载硬件驱动程序；（有可能会借助于ramdisk加载驱动）
        - 以只读方式挂载根文件系统；
        - 运行用户空间的第一个应用程序：`/sbin/init`            
    - 执行 init 程序：
        - CentOS 5：SysV init
            - 配置文件：`/etc/inittab`
        - CentOS 6：Upstart
            - init 的升级版，可以并行启动
            - 配置文件:
            	- `/etc/inittab`: 为向前兼容，基本没哟使用
            	- `/etc/init/\*.conf`
        - CentOS 7：Systemd
            - 配置文件：
            	- `/usr/lib/systemd/system/`
            	- `/etc/systemd/system/`
    - ramdisk：
        - Linux内核的特性之一：使用缓冲和缓存来加速对磁盘上的文件访问；
        - 升级: ramdisk --> ramfs
        - 生成工具:
            - CentOS 5: initrd  -- `mkinitrd`
            - CentOS 6,7: initramfs -- `dracut, mkinitrd`  

### 1.3 总结:系统初始化流程（内核级别）
1. POST自检，
2. 按照BootSequence(BIOS)查找能开机启动的设备
3. 在设备的 MBR上加载 BootLoader，BootLoader 去磁盘分区上读取内核。
4. Kernel可能会借助于 ramdisk 加载真正根文件系统所在设备的驱动程序
5. 内核装载 rootfs（readonly，并执行开机启动程序 `/sbin/init`


需要说明的是无论是下述的 ramdisk 还是 BootLoader 都是在安装操作系统时针对当前硬件生成的。所以 BootLoader 是能够识别当前主机的硬盘设备的。但是需要注意的是BootLoader 是需要和磁盘分区打交道的，而BootLoader 本身一般是无法驱动那些软设备，逻辑设备(LVM),也无法驱动RAID这些复杂的逻辑结构，因此内核只能放在基本的磁盘分区上。
