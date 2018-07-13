# 16.1 systemd及systemctl
本节我们学习 Centos7 的开机启动程序 Systemd，及其服务管理工具 systemctl。本节内容如下:
1. Systemd 概述
2. Systemctl 命令的使用
3. Systemd 配置文件格式

## 1. Sysmted 概述：
MBR 架构的系统，开机启动过程是 `POST --> Boot Sequeue(BIOS) --> Bootloader(MBR) --> Kernel(ramdisk) --> rootfs --> /sbin/init`，而 Systemd 正是 Centos7 的/sbin/init 程序

### 1.2  Systemd的新特性
systemd 相比于 Centos5 的 SysV init，和 Centos 的 Upstart，具有如下特性:
1. 新特性
	- 系统引导时实现服务并行启动；
	- 按需激活进程；
	- 系统状态快照；
	- 基于依赖关系定义服务控制逻辑；
2. 关键特性：
	- 基于socket的激活机制：socket与程序分离；
	- 基于bus的激活机制；
	- 基于device的激活机制；
	- 基于Path的激活机制；
	- 系统快照：保存各unit的当前状态信息于持久存储设备中；
	- 向后兼容sysv init脚本；/etc/init.d/
	- 不兼容：
		- systemctl的命令是固定不变的；
		- 非由systemd启动的服务，systemctl无法与之通信；

#### 1.2 服务配置
Sysv init 和 Upstart 中，服务的管理单元是一个个具有特定格式的 shell 脚本，由 service 命令统一进行管理。而 Systemd 中服务的核心单元叫 Unit，unit 由其相关配置文件进行标识、识别和配置，配置文件中主要包含了系统服务、监听的socket、保存的快照以及其它与init相关的信息。systemd 按照功能将 unit 分为了如下几种类型。

#### unit的常见类型
|类型|文件扩展名|作用|
|:---|:---|:---|
|Service unit|.service|用于定义系统服务|
|Target unit|.target|用于模拟实现运行级别|
|Device unit|.device|用于定义内核识别的设备|
|Mount unit|.mount|定义文件系统挂载点|
|Socket unit|.socket|用于标识进程间通信用到的socket文件|
|Snapshot unit|.snapshot|管理系统快照|
|Swap unit|.swap|用于标识swap设备|
|Automount unit|.automount|文件系统自动点设备|
|Path unit|.path|用于定义文件系统中的一文件或目录|

#### systemd 的配置文件
systemd 的配置文件位于以下三个目录中
- `/usr/lib/systemd/system`
- `/run/systemd/system`
- `/etc/systemd/system`


## 2. systemctl 命令使用
### 2.1 管理系统服务 (service unit)

`systemctl  [OPTIONS...]  COMMAND  [NAME...]`

#### 服务启动与关闭
|作用|init|systemctl|
|:---|:---|:---|
|启动|service  NAME  start | systemctl  start  NAME.service|
|停止|service  NAME  stop|systemctl  stop  NAME.service|
|重启| service  NAME  restart | systemctl  restart  NAME.service|
|状态| service  NAME  status |systemctl  status  NAME.service|
|条件式重启|service  NAME  condrestart|systemctl  try-restart  NAME.service|
|重载或重启服务||systemctl  reload-or-restart  NAME.servcie|
|重载或条件式重启服务||systemctl  reload-or-try-restart  NAME.service|

#### 服务状态查看
|作用|init|systemctl|
|:---|:---|:---|
|查看某服务当前激活与否的状态|| systemctl  is-active  NAME.service|
|查看所有已激活的服务||systemctl  list-units  --type  service|
|查看所有服务（已激活及未激活)|| systemctl  list-units  -t  service  --all|

#### 开机自启
|作用|init|systemctl|
|:---|:---|:---|
|设置服务开机自启|chkconfig  NAME  on|systemctl  enable  NAME.service|
|禁止服务开机自启|chkconfig  NAME  off |systemctl  disable  NAME.service|
|查看某服务是否能开机自启|chkconfig  --list  NAME|  systemctl  is-enabled  NAME.service|
|查看所有服务的开机自启状态|chkconfig  --list|  systemctl  list-unit-files --type service|
|禁止某服务设定为开机自启||systemctl  mask  NAME.service|
|取消此禁止||systemctl  unmask  NAME.servcie|

#### 依赖关系
|作用|init|systemctl|
|:---|:---|:---|
|查看服务的依赖关系||systemctl  list-dependencies  NAME.service|

### 2.2 管理 target units
|作用|init|systemctl|
|:---|:---|:---|
|运行级别|0|runlevel0.target,  poweroff.target|
|运行级别|1| runlevel1.target,  rescue.target|
|运行级别|2| runlevel2.tartet,  multi-user.target|
|运行级别|3| runlevel3.tartet,  multi-user.target|
|运行级别|4|runlevel4.tartet,  multi-user.target|
|运行级别|5| runlevel5.target,  graphical.target|
|运行级别|6| runlevel6.target,  reboot.target|
|级别切换|init  N | systemctl  isolate  NAME.target|
|查看级别| runlevel |systemctl  list-units  --type  target|
|查看所有级别| |systemctl  list-units  -t  target  -a|
|获取默认运行级别|/etc/inittab|systemctl  get-default  |
|修改默认运行级别|/etc/inittab |systemctl  set-default   NAME.target|
|切换至紧急救援模式|| systemctl  rescue|
|切换至emergency模式|| systemctl  emergency|


### 2.3 其它常用快捷命令
- 关机： `systemctl  halt`,  `systemctl  poweroff`
- 重启： `systemctl  reboot`
- 挂起： `systemctl  suspend`
- 快照： `systemctl  hibernate`
- 快照并挂起： `systemctl  hybrid-sleep`


## 3. service unit file 配置
### 3.1 unit file 组成
unit file 通常由如下 三个部分组成:
1. [Unit]：
	- 定义与Unit类型无关的通用选项；
	- 用于提供 unit 的描述信息、unit 行为及依赖关系等；
2. [Service]：
	- 与特定类型相关的专用选项；此处为 Service 类型；
3. [Install]：
	- 定义由`systemctl  enable`以及`systemctl  disable`命令在实现服务启用或禁用时用到的一些选项；

```
# httpd.service
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/httpd
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
ExecStop=/bin/kill -WINCH ${MAINPID}
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

#### Unit段
- `Description`：描述信息； 意义性描述；
- `After`：定义unit的启动次序；表示当前unit应该晚于哪些unit启动；其功能与Before相反；
- `Requies`：依赖到的其它units；强依赖，被依赖的units无法激活时，当前unit即无法激活；
- `Wants`：依赖到的其它units；弱依赖；
- `Conflicts`：定义units间的冲突关系；

#### Service段
- `Type`：用于定义影响ExecStart及相关参数的功能的unit进程启动类型；
	- simple：
	- forking：
	- oneshot：
	- dbus：
	- notify：
	- idle：
- `EnvironmentFile`：环境配置文件；
- `ExecStart`：指明启动unit要运行命令或脚本；
- `ExecStartPre`
- `ExecStartPost`
- `ExecStop`：指明停止unit要运行的命令或脚本；
- `Restart`：

#### Install段
- `Alias`：
- `RequiredBy`：被哪些units所依赖；
- `WantedBy`：被哪些units所依赖；

### 3.2 重载 unit 文件
对于新创建的unit文件或修改了的unit文件，要通知systemd重载此配置文件 `systemctl  daemon-reload`
