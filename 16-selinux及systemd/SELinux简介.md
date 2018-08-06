# 16.2 SELinux简介
对于安全性很多人存在误解，觉得 Linux 比 windows 更加安全，其实不然。SELinux(Security-Enhanced Linux) 是美国国家安全局（NSA）对于强制访问控制的实现，用于增强 Linux 的安全性。SELinux 在实际生产环境中使用的很少，原因并不是 SELinux 不够好，而是想要做到精准的权限控制，需要明确知道并管理进程需要访问的资源，对这些信息的管理本身有很大负担。所以本节我们只介绍 SELinux 的简单原理和管理，并不会对其做深入介绍。具体内容包括:
1. SELinux 的权限模型
2. SELinux 工作模型
3. SELinux 管理

## 1. SELinux 权限模型
Linux传统权限模型下，进程能够访问的哪些资源，取决于进程的发启者能够访问的资源集合。这样存在一些弊端，资源所需访问的资源很少，但是能够访问的资源却很大，一旦进程被不怀好意的人控制，就会对 Linux 安全造成威胁。因此 SELinux 才用最小权限法则，进程只能访问那些它必需访问控制的资源，这样就可以提高 Linux 的安全性。两种权限模型的对比如下:
1. Linux传统权限模型
    - 权限模型: DAC (Discretionary Access Control) 自主访问控制
    - 进程权限: 取决于进程发起者作为属主、属组、其它用户的权限集和
2. SELinux:
    - 权限模型:
        - MAC (Mandatory Access Control): 强制访问控制
        - TE (Type Enforcement)：最小权限法则
    - 进程权限: 取决于SELinux 规则库

SELinux 有两种工作级别，不同工作级别下，受控级别的范围不同:
- strict： 每个进程都收到 selinux 的控制
- targeted: 仅有限个进程受到 selinux 的控制，只监控容易被入侵的进程

之所以有 targeted 级别，主要还是受限于管理所有进程能够访问资源的成本太高



## 2. SELinux 工作模型
进程的执行过程可以概括成 "进程对资源执行的操作" 即

`subject operation object`
- subject: 进程主体
- object: 系统资源，主要是文件
- operation: 进程对资源能够执行的操作

SELinux 的核心就是确定"进程能够对哪些资源执行什么操作"。为了将进程与资源关联起来，SELinux 为每个进程及文件提供了安全标签

安全标签: `user:role:type::`
- user: SELinux 的 user
- role: 角色
- type: 类型
  - 进程的 type 称为域 domain,表示一个空间
  - 资源的 type 称为类型，域能访问哪些资源类型取决于 SELinux 规则库 policy
  - domian 包含的 type 即进程能够操作的资源范围，domian 与 type 的对应关系记录在 SELinux 的规则库中

除了对进程访问资源的控制外，SELinux 还对进程的功能作了限制，比如 httpd 进程而言，其有上传和下载等功能，相对于下载而言，上传功能的风险则高的多。因此默认情况下高风险功能在 SELinux 中是禁止的，想要启用必需显示开启。这部分控制又称为 **SELinux 的布尔规则设置**。



### 2.1 进程的安全标签
如下第一段 LABEL 即为进程的域，由 5 段组成，后两段对于我们了解 SELinux 意义不大

```
[root@hp ~]# ps auxZ
LABEL                           USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
system_u:system_r:init_t:s0     root         1  0.7  0.1 194440  9048 ?        Ss   21:28   0:02 /usr/lib/systemd/systemd --switched-root --system --deserialize
system_u:system_r:kernel_t:s0   root         2  0.0  0.0      0     0 ?        S    21:28   0:00 [kthreadd]
system_u:system_r:kernel_t:s0   root         3  0.0  0.0      0     0 ?        S    21:28   0:00 [ksoftirqd/0]
system_u:system_r:kernel_t:s0   root         4  0.0  0.0      0     0 ?        S    21:28   0:00 [kworker/0:0]
```


### 2.2 文件的类型
`system_u:object_r:admin_home_t:s0` 即为文件的类型

```
[root@hp ~]# ll -Z .
-rw-------. root root system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Desktop
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Documents
```


### 2.3 SELinux 规则库 policy
SELinux 的规则遵循“法无授权即禁止不可行”的原则，即如果进程受 SELinux 控制，如果规则库中没有显示定义规则则禁止访问。另外由于所由进程对资源的访问都会读取 SELinux 规则库，因此规则库以二进制格式进行存放，需要专用的命令才能修改。

SELinux 的规则库即按照我们之前所说的模型进行编写:

`subject operation object` ==> `domain --> policy --> type`
- subject: 主-进程  domain
- object: 宾-资源    type
    - Files
    - Directories
    - Porcesses
    - Special files or various types(块设备文件、字符设备、FIFO、socket)
    - FileSystems
    - Links
    - File descriptors
- operation: 谓-操作
    - Create
    - Read
    - Write
    - Lock
    - Rename
    - Link
    - Unlink
    - Append
    - Excute
    - I/O Control

### 2. SELinux 配置文件
SELinux 的配置位于 `/etc/sysconfig/selinux`

```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
参数:
1. SELINUXTYPE: SELinux 的工作级别
2. SELINUX: SELinux 启用状态
    - disabled: 禁用，关闭 SELinux
    - enforcing: 启用，强制，一旦进程不符合 SELinux 的权限控制会禁止进程访问相关资源
    - permissive: 启用，警告，SELinux 不会禁止进程违规访问资源，仅记录日志
3. 附注: SELinux 日志文件则位于: `/var/log/audit/audit.log`

需要特别说明的是由 `disabled` --> `enforcing|permissive` 需要重启系统才会生效，因为系统要为所有受控的进程和文件打上安全标签

## 3. SELinux 相关命令
### 3.1 SELinux 启用状态管理
#### getenforce
`getenforce`
- 作用: 获取当前 SELinux 状态

```
[root@hp ~]# getenforce
Permissive
```
显示:
- disabled: 禁用
- permissive: 警告，仅记录日志
- enforcing: 强制

#### setenforce
`setenforce value`
- 作用: 启用SELinux
- value:
  - `0`: 设置为 permissive
  - `1`: 设置为 enforcing
- 效力: 当前有效，开机后无效
- 附注: 永久有效，需修改配置文件。

需要特别注意的是使用 setenfoce 命令的前提是 SELinux 状态不能为 disabled。如果 SELinux 为 disabled 只能修改配置文件然后重启。

```bash
vim /etc/selinux/config   # 或 /etc/sysconfig/selinux
SELINUX={disabled|enforcing|permissive}
```

### 2.3 SELinux type 标签管理
`ls -Z /path/to/somefile`
- 作用: 查看文件标签

`ps auxZ`
- 作用: 查看进程标签


#### chcon
`chcon OPTIIONS file`
- 作用: change context 修改文件安全标签
- OPTIONS
  - `-t TYPE`: 设置文件 type
  - `-R`: 递归修改
  - `--reference=file`: 参考某文件的标签进行设置

```
[root@hp tmp]# ll -Z aa
-rw-r--r--. root root unconfined_u:object_r:user_tmp_t:s0 aa

[root@hp tmp]# chcon -t admin_home_t aa
[root@hp tmp]# ll -Z aa
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 aa

[root@hp tmp]# chcon aa --reference bb
[root@hp tmp]# ll -Z aa
-rw-r--r--. root root unconfined_u:object_r:user_tmp_t:s0 aa

```

#### restorecon
`restorecon -R file`
- 作用: 还原默认标签
- `-R`: 递归修改


### 2.4 SELinux的布尔规则设置
#### getsebool
`getsebool [-a] [boolean_name]`
- 作用: 显示 SELinux 布尔型规则
- 参数: boolean_name 规则名称
- 选项: `-a` 显示所有布尔型规则

```
[root@hp ~]# getsebool -a|grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off
httpd_can_network_memcache --> off
```

#### setsebool
`setsebool [ -PNV ] boolean value | bool1=val1 bool2=val2 ...`
- 作用: 设置布尔规则
- VARIABLE:
  - `={0|off|false}`: 关闭功能
  - `={1|on|true}`: 开启功能
- 选项:
  - `P`: 将修改写入配置文件中，否则仅仅当前设置有效

```
[root@hp tmp]# getsebool httpd_use_nfs
httpd_use_nfs --> off

[root@hp tmp]# setsebool httpd_use_nfs 1
[root@hp tmp]# getsebool httpd_use_nfs
httpd_use_nfs --> on
```
