# 16.1 systemd及systemctl
本节我们学习 Centos7 的开机启动程序 Systemd，及其服务管理工具 systemctl。我们会与 Centos6 中的 upstart 的启动程序对比来讲解。大家也可以参考[阮一峰老师的博客](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)。本节内容如下:
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
- `/usr/lib/systemd/system`: 实际配置文件的存存放位置
- `/run/systemd/system`：不常用
- `/etc/systemd/system`: 基本上都是软连接

对于那些支持 Systemd 的软件，安装的时候，会自动在`/usr/lib/systemd/system`目录添加一个配置文件。
如果你想让该软件开机启动，就执行下面的命令（以httpd.service为例）。

```
[root@hp system]# ll /etc/systemd/system/default.target
lrwxrwxrwx. 1 root root 40 3月   5 17:37 /etc/systemd/system/default.target -> /usr/lib/systemd/system/graphical.target

[root@hp system]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

上面的命令相当于在 `/etc/systemd/system` 目录添加一个符号链接，指向 `/usr/lib/systemd/system` 里面的httpd.service文件。
这是因为开机时，Systemd只执行 `/etc/systemd/system` 目录里面的配置文件。这也意味着，如果把修改后的配置文件放在该目录，就可以达到覆盖原始配置的效果。

除了使用普通的文本查看命令外查看配置文件外，`systemctl cat NAME.service` 可通过服务名称直接查看配置文件

```bash
[root@hp system]$ systemctl cat httpd
# /usr/lib/systemd/system/httpd.service
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
# We want systemd to give httpd some time to finish gracefully, but still want
# it to kill httpd after TimeoutStopSec if something went wrong during the
# graceful stop. Normally, Systemd sends SIGTERM signal right after the
# ExecStop, which would kill httpd. We are sending useless SIGCONT here to give
# httpd time to finish.
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```


## 2. systemctl 命令使用
### 2.1 管理系统服务 (service unit)

`systemctl  [OPTIONS...]  COMMAND  [NAME...]`
- OPTIONS:
	- `-t, --type=`: 指定查看的 unit 类型
	- `-a, --all`：查看所由服务

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

```
[root@hp system]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2018-08-07 09:14:30 CST; 1s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 6170 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd.service
           ├─6170 /usr/sbin/httpd -DFOREGROUND
           ├─6174 /usr/sbin/httpd -DFOREGROUND
           ├─6176 /usr/sbin/httpd -DFOREGROUND
           ├─6177 /usr/sbin/httpd -DFOREGROUND
           ├─6178 /usr/sbin/httpd -DFOREGROUND
           ├─6180 /usr/sbin/httpd -DFOREGROUND
           └─6181 /usr/sbin/httpd -DFOREGROUND

8月 07 09:14:28 hp.tao systemd[1]: Starting The Apache HTTP Server...
8月 07 09:14:30 hp.tao systemd[1]: Started The Apache HTTP Server.
```
输出:
- Loaded行：配置文件的位置，是否设为开机启动
- Active行：表示正在运行
- Main PID行：主进程ID
- Status行：由应用本身（这里是 httpd ）提供的软件当前状态
- CGroup块：应用的所有子进程
- 日志块：应用的日志


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

```bash
# systemctl cat sshd
# /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

#### Unit段
- `Description`：当前服务的简单描述
- `After`：定义unit的启动次序；表示当前unit应该晚于哪些unit启动；其功能与Before相反；
- `Before`：定义sshd.service应该在哪些服务之前启动
- `Requies`：依赖到的其它units；强依赖，被依赖的units无法激活或异常退出时，当前unit即无法激活；
- `Wants`：依赖到的其它units；弱依赖，被依赖的units无法激活时，不影响当 unit 的启动；
- `Conflicts`：定义units间的冲突关系；
- 附注：
	- After和Before字段只涉及启动顺序，不涉及依赖关系
	- Wants字段与Requires字段只涉及依赖关系，与启动顺序无关，默认情况下是同时启动的

#### Service段
- `Type`：用于定义影响ExecStart及相关参数的功能的unit进程启动类型；
	- `simple`（默认值）：ExecStart字段启动的进程为主进程
	- `forking`：ExecStart字段将以fork()方式启动，此时父进程将会退出，子进程将成为主进程
	- `oneshot`：类似于simple，但只执行一次，Systemd 会等它执行完，才启动其他服务
	- `dbus`：类似于simple，但会等待 D-Bus 信号后启动
	- `notify`：类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务
	- `idle`：类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合
- `EnvironmentFile`：指定当前服务的环境参数文件
- `ExecStart`：指明启动unit要运行命令或脚本；其中的变量$OPTIONS就来自EnvironmentFile字段指定的环境参数文件
- `ExecReload`：重启服务时执行的命令
- `ExecStop`：停止服务时执行的命令
- `ExecStartPre`：启动服务之前执行的命令
- `ExecStartPost`：启动服务之后执行的命令
- `ExecStopPost`：停止服务之后执行的命令
- `Restart`：定义了服务退出后，Systemd 的重启方式
	- `no`（默认值）：退出后不会重启
	- `on-success`：只有正常退出时（退出状态码为0），才会重启
	- `on-failure`：非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启
	- `on-abnormal`：只有被信号终止和超时，才会重启
	- `on-abort`：只有在收到没有捕捉到的信号终止时，才会重启
	- `on-watchdog`：超时退出，才会重启
	- `always`：不管是什么退出原因，总是重启
	- 附注: 对于守护进程，推荐设为on-failure。对于那些允许发生错误退出的服务，可以设为on-abnormal
- `KillMode`:定义 Systemd 如何停止服务
	- `control-group`（默认值）：当前控制组里面的所有子进程，都会被杀掉
	- `process`：只杀主进程
	- `mixed`：主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
	- `none`：没有进程会被杀掉，只是执行服务的 stop 命令。
- `RestartSec`：表示 Systemd 重启服务之前，需要等待的秒数

对于 sshd 服务而言将KillMode设为process，表示只停止主进程，不停止任何sshd 子进程，即子进程打开的 SSH session 仍然保持连接。这个设置不太常见，但对 sshd 很重要，否则你停止服务的时候，会连自己打开的 SSH session 一起杀掉。Restart设为on-failure，表示任何意外的失败，就将重启sshd。如果 sshd 正常停止（比如执行systemctl stop命令），它就不会重启。


所有的启动设置之前，都可以加上一个连词号（-），表示"抑制错误"，即发生错误的时候，不影响其他命令的执行。比如，`EnvironmentFile=-/etc/sysconfig/sshd`（注意等号后面的那个连词号），就表示即使/etc/sysconfig/sshd文件不存在，也不会抛出错误

#### Install段
- `Alias`：
- `RequiredBy`：被哪些units所依赖；
- `WantedBy`：表示该服务所在的 Target

### 3.2 修改配置文件后重启
对于新创建的unit文件或修改了的unit文件，要通知systemd重载此配置文件 `systemctl  daemon-reload`


## 4.Target 的配置文件

```bash
[root@hp system]$ systemctl cat multi-user.target
# /usr/lib/systemd/system/multi-user.target
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
```

Target 配置文件里面没有启动命令
- `Requires`：要求basic.target一起运行。
- `Conflicts`：冲突字段。如果rescue.service或rescue.target正在运行，multi-user.target就不能运行，反之亦然。
- `After`：表示multi-user.target在basic.target 、 rescue.service、 rescue.target之后启动，如果它们有启动的话。
- `AllowIsolate`：允许使用systemctl isolate命令切换到multi-user.target
