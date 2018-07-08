# 14.2 Centos5 init 启动流程
在上一节我们详细讲解了开机启动流程中内核级部分，接下来我们讲学习内核加载并完成根文件系统之后 init 程序的启动过程。因为内容过多，而且不同Centos 版本并不相同，因此我们将分成两个节，本节将讲解一下内容:
1. CentOS5 SysV init 的启动流程
	- `/etc/inittab` 的格式和内容
	- Centos5 的服务启动方式
	- chkconfig 设置服务开机启动
	- `/sbin/init` 程序执行的操作
2. CentOS6 Upstart 的启动流程

## 1.1 CentOS 5： SysV init
### 1.1 运行级别
- 运行级别：为了系统的运行或维护等目的而设定的机制；
    - 0-6：7个级别；
    - 0: 关机, shutdown
    - 1: 单用户模式(single user)，root用户，无须认证；维护模式；
    - 2: 多用户模式(multi user)，会启动网络功能，但不会启动NFS；维护模式；
    - 3: 多用户模式(mutli user)，完全功能模式；文本界面；
    - 4: 预留级别：目前无特别使用目的，但习惯以同3级别功能使用；
    - 5: 多用户模式(multi user)， 完全功能模式，图形界面；
    - 6: 重启，reboot
- 默认级别：3, 5
- 级别切换：`init level`
- 级别查看：
    - `who -r`
    - `runlevel` 

### 1.2 配置文件：/etc/inittab 
`/etc/inittab`
- 作用: 每行定义一种 action 以及与之对应的 process
- 格式: `id:runlevels:action:process` 
    - `id`：一个任务的标识符；
    - `runlevels`：在哪些级别启动此任务，格式如下:
    	- n: 单个数字例如 2，表示仅在第二级别
    	- nnn: 多个数字例如 234，表示在第二三四级别
    	- 也可以为空，表示所有级别；
    - `action`：在什么条件下启动此任务；
    - `process`：任务；
            
**action**：
- wait：等待切换至此任务所在的级别时执行一次；
- respawn：一旦此任务终止，就自动重新启动之；
- initdefault：设定默认运行级别；此时，process省略；
- sysinit：设定系统初始化方式，此处一般为指定`/etc/rc.d/rc.sysinit`脚本；


```            
id:3:initdefault:        -- 设置系统默认启动级别
si::sysinit:/etc/rc.d/rc.sysinit  -- 系统初始化脚本

l0:0:wait:/etc/rc.d/rc  0  -- rc 脚本框架，启动对应级别下的服务
l1:1:wait:/etc/rc.d/rc  1
…………
l6:6:wait:/etc/rc.d/rc  6

tty1:2345:respawn:/usr/sbin/mingetty tty1  ---  虚拟终端启动
... ...
tty6:2345:respawn:/usr/sbin/mingetty tty6    
# mingetty会调用login程序；
# 打开虚拟终端的程序除了mingetty之外，还有诸如getty等；
```
                    
**rc 脚本框架：**
```
for  srv  in  /etc/rc.d/rc#.d/K*; do
    $srv  stop
done

for  srv  in  /etc/rc.d/rc#.d/S*; do
    $srv  start
done
```
- rc脚本：接受一个运行级别数字为参数；
- rc 3: 意味着去启动或关闭/etc/rc.d/rc3.d/目录下的服务脚本所控制服务；
- K\*：要停止的服务；K\#\#\*，优先级，数字越小，越是优先关闭；依赖的服务先关闭，而后关闭被依赖的；
- S\*：要启动的服务；S\#\#\*，优先级，数字越小，越是优先启动；被依赖的服务先启动，而依赖的服务后启动；
- rc\#.d/ 目录下所有文件都是链接文件，连接到 /etc/rc.d/init.d/ 

```
> ls /etc/rd.d/
init.d rc rc0.d  rc1.d  rc2.d  rc3.d rc4.d rc5.d  rc6.d  rc.local  rc.sysinit    
```

### 1.2 服务启动
`/etc/init.d/* (/etc/rc.d/init.d/*)`脚本执行方式：
- `/etc/init.d/SRV_SCRIPT  {start|stop|restart|status}`
- `service     SRV_SCRIPT  {start|stop|restart|status}`

### 1.3 配置服务开机启动
chkconfig命令：
- 作用: 管控/etc/init.d/每个服务脚本在各级别下的启动或关闭状态；
- 添加：`chkconfig  --add  name`
	- 作用: 将 name 脚本添加到service 命令的控制中，并按照脚本中 chkconfig 的配置在对应级别下设置开机启动，其他级别下设置开机关闭
- 启动/关闭指定级别服务： 
    - `chkconfig  [--level  LEVELS]  name  on|off|reset` 
    - `--level LEVELS`：指定要控制的级别；默认为2345；
- 查看：`chkconfig  --list  [name]`
- 删除：`chkconfig  --del  name`
    - 作用: 将服务从 service 管理的范围内删除，并从各个级别的开机启动中删除
- 注意：正常级别下，最后启动的一个服务S99local 没有链接至/etc/init.d下的某脚本，而是链接至了/etc/rc.d/rc.local （/etc/rc.local）脚本；因此，不便或不需写为服务脚本的程序期望能开机自动运行时，直接放置于此脚本文件中即可

```
> vim /etc/init.d/testsrv

#!/bin/bash
# testsrv  serviec testing script
# 
# chkconfig: 234  50 60
# description:  testing service


# 注解: 必须要有 chkconfig:
# chkconfig: LLL  NN NN  -- 被添加到的级别，启动优先级，关闭优先级
# chkconfig: - 50 60     -- 表示没有级别

$prog=$(basename $0)

if [ $# -lt 1 ]; then
	echo "Usage: ${prog} {start|stop|status|restart}"
	exit 1
fi

if [ "$1" == "start" ]; then
	echo "start $prog success"
elif [ "$1" == "stop" ]; then
	echo "stop $prog success"
elif [ "$1" == "status" ]; then
	if pidof $prog &> /dev/null; then
		echo "$prog is running"
	else
		echo "$prog is stopped"
	fi
elif [ "$1" == "restart" ]; then
	echo "restart $prog success"
else
	echo "Usage: ${prog} {start|stop|status|restart}"
	exit 2
fi



> chkconfig --add testsrv
> ls /etc/rc.d/rc3.d/ |grep testsrv

> chkconfig --list testsrv
> chkconfig --level 23 testsrv off
> chkconfig --del testsrv
```


### 1.5 总结（用户空间的启动流程）： 
`/sbin/init (/etc/inittab)` 
1. 设置默认运行级别 
3. 运行系统初始化脚本(`/etc/rc.d/rc.sysinit`)，完成系统初始化 
	- 设置主机名
	- 设置欢迎信息
	- 激活udev和selinux
	- **挂载/etc/fstab文件中定义的所有文件系统**
	- **检测根文件系统，并以读写方式重新挂载根文件系统**
	- 设置系统时钟；
	- **根据/etc/sysctl.conf文件来设置内核参数**
	- 激活lvm及软raid设备
	- 激活swap设备
	- 加载额外设备的驱动程序
	- 清理操作 
4. 关闭对应级别下需要停止的服务，启动对应级别下需要开启的服务
5. 设置登录终端 
6. [--> 启动图形终端]
    

### 2. CentOS 6：
Centos 6 中init程序为 upstart，但依然为/sbin/init 配置文件包括如下:
    - /etc/init/\*.conf
    - /etc/inittab（仅用于定义默认运行级别）
    - 注意：\*.conf为upstart风格的配置文件，需遵循其配置语法；
        - rcS.conf
        - rc.conf
        - start-ttys.conf

CentOS 6启动流程：
- POST --> Boot Sequence(BIOS) --> Boot Loader (MBR) --> Kernel(ramdisk) --> rootfs --> switchroot --> /sbin/init -->(/etc/inittab, /etc/init/\*.conf) --> 设定默认运行级别 --> 系统初始化脚本 --> 关闭或启动对应级别下的服务 --> 启动终端
