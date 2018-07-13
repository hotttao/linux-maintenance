# 15.1 Linux内核模块功能定制

## 1. 编译内核
- 程序包的编译安装：./configure, make, make install
- 前提：开发环境（开发工具，开发库），头文件：/usr/include
    - 准备好开发环境；
    - 获取目标主机上硬件设备的相关信息；
    - 获取到目标主机系统功能的相关信息，例如要启用的文件系统；
    - 获取内核源代码包：www.kernel.org

### 1.1 准备开发环境：
CentOS 6：
- 包组：
    - Development Tools
    - Server Platform Development

CentOS 7：
- 包组：
    - Development Tools
    - Server Platform Development
- 包：
    - ncurses-devel

### 1.2 获取目标主机上硬件设备的相关信息：
- CPU：
    - cat  /proc/cpuinfo
    - lscpu
    - x86info -a
- PCI设备：
    - lspci
        - -v
        - -vv
    - lsusb: usb 信息
        - -v
        - -vv
    - lsblk: 块设备信息
- 了解全部硬件设备信息：
    - hal-device

## 2. 内核编译过程：
### 2.1 简单依据模板文件的制作过程：
- tar  xf  linux-3.10.67.tar.xz  -C  /usr/src
- cd  /usr/src
- ln  -sv  linux-3.10.67  linux
- cd  linux
- cp  /boot/conofig-$(uname -r)  ./config
- make menuconfig          配置内核选项
- make  [-j \#]            编译内核，可使用-j指定编译线程数量
- make modules_install    安装内核模块
- make install            安装内核
    - 安装 bzImage 为 /boot/vmlinuxz-VERSION-RELEASE
    - 生成 initramfs 文件
    - 编辑 grub 的配置文件
- 重启系统，选择使用新内核；

### 2.2 screen命令：
- 作用: 终端模拟器，允许在一个终端上打开多个屏幕
- 特性: screen 的模拟终端不会因为当前物理终端断开连接而丢失，即 screen 内运行的程序不会因为物理终端断开连接而终止
- 选项:
    - 打开screen：  screen
    - 拆除screen：  Ctrl+a, d
    - 列出screen：  screen  -ls
    - 连接至screen：screen  -r  SCREEN_ID
    - 关闭screen:  exit

### 2.3 编译过程的详细说明：
1. 配置内核选项
    - 支持“更新”模式进行配置：在已有的.config文件的基础之上进行“修改”配置；
        - make config：基于命令行以遍历的方式去配置内核中可配置的每个选项；
        - make menuconfig：基于cureses的文本配置窗口；
        - make gconfig：基于GTK开发环境的窗口界面；  包组“桌面平台开发”
        - make xonfig：基于QT开发环境的窗口界面；
    - 支持“全新配置”模式进行配置：
        - make  defconfig：基于内核为目标平台提供的“默认”配置为模板进行配置；
        - make  allnoconfig：所有选项均为“no”；
2. 编译
    - 多线程编译：make  [-j \#]
    - 编译内核中的一部分代码：
        - 只编译某子目录中的相关代码：
            - cd  /usr/src/linux
            - make  path/to/dir/  -- 只能在内核源码目录内，基于相对路径编译
        - 只编译一个特定的模块
            - cd  /usr/src/linux
            - make  path/to/dir/file.ko
    - 如何交叉编译：
        - 目标平台与当前编译操作所在的平台不同；
        - make  ARCH=arch_name
    - 要获取特定目标平台的使用帮助：                    
        - make  ARCH=arch_name help
3. 如何在执行过编译操作的内核源码树上做重新编译：
    - 事先清理操作：
        - make clean：清理编译生成的绝大多数文件，但会保留config，及编译外部模块所需要的文件；
        - make mrproper：清理编译生成的所有文件，包括配置生成的config文件及某些备份文件；
        - make distclean：相当于mrproper，额外清理各种patches以及编辑器备份文件；
