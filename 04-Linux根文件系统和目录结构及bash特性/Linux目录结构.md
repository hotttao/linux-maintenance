# 4.1 Linux目录机构
文件系统同样是很复杂的东西，具体原理后面会介绍。当前，只要知道文件系统是操作系统对磁盘的抽象，为用户提供了管理磁盘文件的接口。开机启动时，内核加载完毕之后，内核就会挂载用户在开机启动配置文件中设置的根文件系统。Linux 上的文件系统必需挂载到根文件系统上才能被使用。开机启动流程，文件系统会在之后详细介绍。当前我们需要重点了解的是，Linux 上目录和文件的组织结构。


如同在 Windows 上创建目录和文件上一样，我们可以在Linux 上随意的创建和删除文件。但是就像我们很难在别人的Windows 系统上查找文件一样，如果各Linux 发行厂商随意的组织 Linux 的文件，当我们更换一个 Linux 发行版时，我们可能就很难找到配置文件，应用程序；程序开发者也很难统一配置程序的安装目录。所以 Linux 标准委员会为避免这种情况发生，指定了一个标准，叫 FHS(filesystem hierarchy standard)。


FHS 主要对 `/`, `/usr`, `/var` 的使用进行了规范，我们将按照这三个层次进行介绍。在本文的结尾，我们将对Linxu 上的文件系统类型作详细介绍。

## 1. FHS 简介
### 1.1 官方文档简介
**This standard enables:**
- Software to predict the location of installed files and directories, and
- Users to predict the location of installed files and directories.

**We do this by:**
- Specifying guiding principles for each area of the filesystem,
- Specifying the minimum files and directories required,
- Enumerating exceptions to the principles, and
- Enumerating specific cases where there has been historical conflict.

**The FHS document is used by:**
- Independent software suppliers to create applications which are FHS compliant, and work with distributions
which are FHS complaint,
- OS creators to provide systems which are FHS compliant, and
- Users to understand and maintain the FHS compliance of a system.
The FHS document has a limited scope:
- Local placement of local files is a local issue, so FHS does not attempt to usurp system administrators.
- FHS addresses issues where file placements need to be coordinated between multiple parties such as local
sites, distributions, applications, documentation, etc.

### 1.2 FHS 标准内容概述
`/`, `/usr`, `/var` 必需包含的目录，及目录作用如下:

`/`
- `bin/`: 所有用户可用的基本命令程序文件
- `sbin/`: 供系统管理使用的工具程序
- `boot/`: 引导加载器必须用到的各静态文件，包括 kernal, initramfs(initrd), grub 等
- `dev/`: 存储特殊文件或设备文件，设备包括如下两种类型
	- 字符设备，又称线性设备
	- 块设备，又称随机设备
- `etc/`: 系统程序的配置文件，只能为静态
- `home/`: 普通用户家目录的集中位置
- `root/`: 管理员家目录
- `lib`: 为系统启动或根文件系统上的应用程序(/bin, /sbin等)提供共享库，以及为内核提供内核模块
	- libc.so.\*： 动态链接的 C库
	- ld\*: 运行时链接器或加载器
	- modules/: 用于存储内核模块的目录
- `lib64`: 同lib，64 位系统特有的存放 64 位共享库的目录
- `media/`: 便携式设备挂载点
- `mnt/`: 其他文件系统的临时挂载点
- `opt/`: 附加应用程序的安装位置，可选路径
- `srv/`: 当前主机为服务提供的数据
- `tmp`: 为哪些会产生临时文件的程序提供的用于存储临时文件的目录
- `usr/`: shareable, read-only data，独立的层级目录，存放全局共享的只读数据路径
	- `bin/`:
	- `sbin/`: 非管理或维护系统运行所必须的，额外添加的管理命令
	- `lib`:
	- `lib64`:
	- `includ/`: C 程序头文件
	- `share/`: 命令手册页和自带文档等框架特有的文件的存储位置
	- `X11R6`: X-Window 程序的安装位置
	- `src`: 程序源码文件的存储位置
	- `local/`: Local hierarchy 独立的层级目录，让系统管理员安装本地应用程序，也通常用于安装第三方程序
  		- 应用程序多版本共存时，新版程序通常安装于此目录
  		- 层级结构与 /usr 类似
- `var/`: var Hierarchy, 独立的层级目录，用于存储常发生变化的数据的目录
	- `cache/`: Application cache data
	- `lib/`: Variable state information
	- `local/`: Variable data for /usr/local
	- `lock/`: Lock files
	- `log`: Log files and directories
	- `opt/`: Variable data for /opt
	- `run/`: Data relevant to running processes
	- `spool/`: Application spool data
	- `tmp/`: Temporary files preserved between system reboots
- `proc/`:
	- 基于内存的虚拟文件系统，用于为内核及进程存储其相关信息；
	- 它们多为内核参数，例如net.ipv4.ip_forward, 虚拟为net/ipv4/ip_forward, 存储于/proc/sys/, 因此其完整路径为/proc/sys/net/ipv4/ip_forward；
- `sys/`:
	- 用于挂载sysfs虚拟文件系统
	- 提供了一种比proc更为理想的访问内核数据的途径；
	- 其主要作用在于为**管理Linux设备**提供一种统一模型的的接口；
	- 参考: https://www.ibm.com/developerworks/cn/linux/l-cn-sysfs/

## 2. / 根目录

## 3. /usr 目录

## 4. /var 目录

## 5. Linux 系统上的文件类型
### 5.1 常见文件类型：
```
> ll
drwxrwxr-x.  2 tao tao  158 2月  25 18:32 anki
drwxrwxr-x.  3 tao tao   43 2月  24 18:36 coding
drwxrwxr-x.  4 tao tao   53 1月  30 14:11 linux
```
`ls -l` 命令显示结果第一列的首子母即表示文件类型，Linux 中的文件类型如下
- `-`：常规文件；即f；
- `d`: directory，目录文件(路径映射)
- `b`: block device，块设备文件，支持以“block”为单位进行随机访问
- `c`：character device，字符设备文件，支持以“character”为单位进行线性访问
- `l`：symbolic link，符号链接文件
- `p`: pipe，命名管道
- `s`: socket，套接字文件


### 5.2 设备文件的设备号
```
> ll /dev
# 10, 58 表示设备的设备号
crw-------. 1 root root     10,  58 6月  19 21:35 network_latency
crw-------. 1 root root     10,  57 6月  19 21:35 network_throughput
```
设备文件还有设备号，其作用如下:
- major number：主设备号，用于标识设备类型，进而确定要加载的驱动程序; 8位二进制：0-255
- minor number：次设备号，用于标识同一类型中的不同的设备; 8位二进制：0-255
