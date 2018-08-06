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


### 1.1 SELinux 配置文件
SELinux 的配置位于 `/etc/sysconfig/selinux`
```
```

状态: SELinux 根据启用与否有三个状态
    - disabled: 禁用，关闭 SELinux
    - enforcing: 启用，强制，一旦进程不符合 SELinux 的权限控制会禁止进程访问相关资源
    - permissive: 启用，警告，SELinux 不会禁止进程违规访问资源，仅记录日志


## 2. SELinux 工作模型
进程的执行过程可以概括成 `subject operation object`
- subject: 进程主体
- object: 系统资源 执行了什么操作，资源主要是文件
- operation: 进程对资源能够执行的操作

即 SELinux 的核心就是确定"进程能够对哪些资源执行什么操作"。

为了将进程与资源关联起来，SELinux 将
1. 进程标记为 domian(域) 表示一个空间
2. 将资源标记为 type(类型)
3. domian 包含的 type 即进程能够操作的资源范文，domian 与 type 的对应关系记录在 SELinux 的规则库中


### 2.1 进程的域
```
ps auxZ

```


### 2.2 文件的类型
```
ll -Z .

```

1. SELinux 为每个文件提供安全标签，也为每个进程提供安全标签
2. 安全标签: user:role:type
    - user: SELinux 的 user
    - role: 角色
    - type: 类型
3. 进程的 type 称为域 domain，资源的 type 称为类型，域能访问哪些资源类型取决于 SELinux 规则库 policy  

#### 1.2.3 SELinux 规则库 policy
- 法则: 主谓宾 subject operation object -- domain --> policy --> type
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

#### 1.2.4 SELinux 日志文件
- 位置: /var/log/audit/audit.log

## 2. SELinux 相关命令
### 2.1 查看 SELinux 状态
getenforce 命令
- disabled: 禁用
- permissive: 警告，仅记录日志
- enforcing: 强制

### 2.2 启用SELinux的方法：
1. setenforce {0|1}
    - 效力: 当前有效，开机后无效
    - 参数:
        - 0: 设置为 permissive
        - 1: 设置为 enforcing
2. 修改配置文件:
    - vim /etc/selinux/config，或/etc/sysconfig/selinux
    - SELINUX={disabled|enforcing|permissive}

### 2.3 SELinux type 标签管理
1. 查看
    - ls -Z /path/to/somefile    user:role:type
    - ps auxZ
2. 更改
    - chcon OPTIIONS file
        - -t TYPE: 设置 type
        - -R: 递归修改
        - --reference=file: 参考某文件的标签进行设置
3. 还原默认标签
    - restorecon -R file:
        - -R: 递归修改

### 2.4 修改SELinux的布尔型规则：
- getsebool [-a] [boolean_name]
    - a: 显示所有布尔型规则
- setsebool VARIABLE={1|0} 或 {on|off} 或 {true|false}
    - P: 将修改写入配置文件中，否则仅仅当前设置有效
