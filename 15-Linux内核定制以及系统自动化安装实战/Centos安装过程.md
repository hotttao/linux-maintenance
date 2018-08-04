# 15.2 Centos安装过程
本节我们来讲解 Centos 系统的安装过程。

## 1. Centos 系统安装
### 1.1 安装程序：anaconda
前面我们说过操作系统的层次，如下图所示，因为直接面向硬件编程是一件非常困难的是，所以才有了操作系统。如果有安装过 Centos 系统就会知道，安装过程有一个操作界面供我们进行选择安装，显然这是一个应用程序，那么这个应用程序是直接在硬件之上编写的么？我们说过在硬件之上编写应用程序是极其困难的，且不易移植，所以我们的安装程序也是构建在内核之上，只不过这个内核不是来自我们的计算机，而是我们的安装光盘或U盘上。Centos 的安装程序就是 annaconda。
```
--------------
|  库调用接口   |
---------------
|   系统调用接口 |
-------------------------------
|    操   作  系   统           |
-------------------------------
|    底   层  硬   件           |
-------------------------------
```

### 1.2 安装光盘的结构
```
mount -r /dev/cdrom /media/cdrom
tree /media/cdrom

```
我们安装光盘的目录结构如上所示，isolinux 就是光盘上操作系统内核所在的目录，其余部分是程序包仓库。

操作系统安装时
1. 首先加载操作系统内核；
  - 光盘安装就是加载位于 isolinux 中的内核
  - 除了光盘，内核还可以来自 U 盘，网络等其他引导设备
  - 通过 PXE 可以实现通过网络自动安装操作系统，这个我们会在后面详述配置过程。
2. 启动 anaconda，进而根据用户选择，安装操作系统
  - anacona及其安装用到的程序包等来自于程序包仓库，此仓库的位置可以为
    - 本地光盘，光盘中 isolinx 之外的就是目录就是程序包仓库
    - 本地硬盘
    - ftp server
    - http server
    - nfs server

anaconda 提供的安装界面分为:
- tui：基于cureses的文本配置窗口
- gui：图形界面


### 1.2 CentOS的安装过程启动流程
当前我们就以光盘安装来讲解 Centos 的安装过程
```
ls /media/cdrom/isolinux

```
1. 加载并启动 BootLoader
  - Stage1: 执行 `isolinux/boot.cat`，光盘的 MBR 包含的就是此文件
  - Stage2: 执行 `isolinux/isolinux.bin` 提供提供安装界面和开机启动菜单
3. BootLoader 引导和加载内核，并装载根文件系统
3. 启动anaconda
    - 默认界面是图形界面：512MB+内存空间；
    - 若需要显式指定启动TUI接口： 向启动内核传递一个参数"text"即可；
        - 方法一： `ESC, boot: linux text`
        - 方法二： `table, text`
    - 如果想手动指定安装仓库：ESC,boot: linux method

#### isolinux.bin
isolinux.bin 其配置文件位于 `isolinux/isolinux.cfg`，配置文件包含了开机启动菜单
```
vim /media/cdrom/isolinux/isolinux.cfg

```

- 每个对应的菜单选项：
    - 加载内核：isolinux/vmlinuz
    - 向内核传递参数：append  initrd=initrd.img .....
