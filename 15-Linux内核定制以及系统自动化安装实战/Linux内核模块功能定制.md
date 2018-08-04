# 15.1 Linux内核模块功能定制
内核编译是一个大工程，需要对硬件，内核各个参数功能都有比较深入了解，才能编译出有特定功能需求的内核。本节主要是带大家了解内核的编译过程，能编译成功即可。本节内容如下:
1. 编译内核的环境准备
2. 根据当前操作系统的编译模板，编译内核

## 1. 编译内核
在编译内核之前，我们需要了解目标主机的功能需求，并准备好开发环境，具体可包括如下几个方面:
1. 准备好开发环境；
2. 获取目标主机上硬件设备的相关信息；
3. 获取到目标主机系统功能的相关信息，例如要启用的文件系统；
4. 获取内核源代码包：http://www.kernel.org

### 1.1 准备开发环境
开发环境主要是准备编译环境，Centos6-7 中安装如下两个包组即可:
1. `Development Tools`: 中文下叫"开发工具"
2. `Server Platform Development`: 中文下叫 "服务器平台开发" - Centos7 可能没有此包组

### 1.2 获取目标主机上硬件设备的相关信息
Linux 中有如下命令，可以帮助我们获取硬件设备的相关信息包括:
1. CPU：
    - `cat  /proc/cpuinfo`
    - `lscpu`
    - `x86info -a`
2. PCI设备：
    - `lspci [-v|-vv]`
    - `lsusb [-v|-vv]`: 显示 usb 信息
    - `lsblk`: 显示块设备信息
3. 了解全部硬件设备信息：`hal-device`(Centos6)

## 2. 内核编译过程：
内核的编译与程序包的编译安装过程类似，遵循`./configure` ==> `make` ==> `make install`。接下来我们将利用现有操作系统的编译安装模板，来编译一个内核。

### 2.1 简单依据模板文件的制作过程：
```bash
#！/bin/bash

# 1. 编译内核
tar  xf  linux-3.10.67.tar.xz  -C  /usr/src
cd  /usr/src
ln  -sv  linux-3.10.67  linux
cd  linux
# cp /boot/config-$(uname -r) .config  # 复制当前系统的编译模板进行参考
make menuconfig          # 配置内核选项
make  [-j \#]            # 编译内核，可使用-j指定编译线程数量
make modules_install     # 安装内核模块
make install             # 安装内核

# make install 会自动完成以下步骤
# 2. 安装 bzImage 为 /boot/vmlinuxz-VERSION-RELEASE
ll arch/x86/boot/bzImage
ll arch/x86_64/boot/bzImage

# 3. 生成 initramfs 文件

# 4. 编辑 grub 的配置文件

# 5. 重启系统，选择使用新内核
```

### 2.2 screen命令：
执行 make 命令时，如果是远程连接到服务器，可能因为网络问题而断开连接，此时 make 就会终止。为了避免因为断开连接导致编译过程前功尽弃，可以使用 screen 命令

#### screen 
- 作用: 终端模拟器，允许在一个终端上打开多个屏幕
- 特性: screen 的模拟终端不会因为当前物理终端断开连接而丢失，即 screen 内运行的程序不会因为物理终端断开连接而终止
- 选项:
    - 打开screen：  `screen`
    - 拆除screen：  `Ctrl+a, d`
    - 列出screen：  `screen  -ls`
    - 连接至screen：`screen  -r  SCREEN_ID`
    - 关闭screen:  `exit`

### 2.3 编译过程的详细说明：
1. 配置内核选项
    - 支持“更新”模式进行配置：在已有的.config文件的基础之上进行“修改”配置；
        - `make config`：基于命令行以遍历的方式去配置内核中可配置的每个选项；
        - `make menuconfig`：基于cureses的文本配置窗口；需要额外安装 `ncurses-devel` 包
        - `make gconfig`：基于GTK开发环境的窗口界面；  包组“桌面平台开发”
        - `make xonfig`：基于QT开发环境的窗口界面；
    - 支持“全新配置”模式进行配置：
        - `make  defconfig`：基于内核为目标平台提供的“默认”配置为模板进行配置；
        - `make  allnoconfig`：所有选项均为“no”；
2. 编译
    - 多线程编译：`make  [-j #]`
    - 编译内核中的一部分代码：
        - 只编译某子目录中的相关代码：
            - `cd  /usr/src/linux`
            - `make  path/to/dir/`  -- 只能在内核源码目录内，基于相对路径编译
        - 只编译一个特定的模块
            - `cd  /usr/src/linux`
            - `make  path/to/dir/file.ko`
    - 如何交叉编译：
        - 目标平台与当前编译操作所在的平台不同；
        - `make  ARCH=arch_name`
    - 要获取特定目标平台的使用帮助：                    
        - `make  ARCH=arch_name help`
3. 如何在执行过编译操作的内核源码树上做重新编译：
    - 事先清理操作：
        - `make clean`：清理编译生成的绝大多数文件，但会保留config，及编译外部模块所需要的文件；
        - `make mrproper`：清理编译生成的所有文件，包括配置生成的config文件及某些备份文件；
        - `make distclean`：相当于mrproper，额外清理各种patches以及编辑器备份文件；
