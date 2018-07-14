# 16.2 SELinux简介
## 1. SELinux
SELinux: Security Enhanced Linux
### 1.1 Linux 权限模型
1. Linux传统权限模型
    - 属主、属组、其它
    - DAC (Discretionary Access Control)： 自主访问控制
2. SELinux:
    - MAC (Mandatory Access Control)
    - TE (Type Enforcement)：最小权限法则
    - 两种工作级别:
        - strict： 每个进程都收到 selinux 的控制
        - targeted: 仅有限个进程受到 selinux 的控制，只监控容易被入侵的进程

### 1.2 SELinux
#### 1.2.1 SELinux 状态
getenforce 命令
    - disabled: 禁用
    - permissive: 警告，仅记录日志
    - enforcing: 强制

#### 1.2.2 工作模型
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
