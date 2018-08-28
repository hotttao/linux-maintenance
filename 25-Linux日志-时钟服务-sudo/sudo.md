# 25.3 sudo
sudo是linux系统管理指令，它允许用户临时以其他用户(通常是root)执行一些或全部指令，其实现的是一种授权机制。普通用户想执行root 用户的特权命令时，可以使用 `su` 切换到管理员，但是这样做有两个坏处，一是 root 用户会被普通用户知道，二是普通用户切换为 root 后获取的是 root 的所有权限，这些都存在安全风险。而 sudo 可以实现授权普通用户执行部分或全部命令，同时无需 root 密码。本节我们就来介绍 sudo 的使用，内容如下:
1. su 用户切换
2. sudo 配置
3. sudo 命令使用

## 1. su 用户切换
`su [OPTION]... [-] [USER [ARG]...]`
- 作用: 用户切换
- 参数: USER，可省，默认为 root
- 选项:
  - `-l`: 交互式登录shell进程，`su -l user` == `su - user`
  - `-c  'COMMAND'`: 不切换用户只执行命令后，并退出

有关交互式登陆，可以回看 [6.10 bash配置文件](06-shell脚本编程/bash的配置文件.md)

## 2. sudo 配置
sudo 能够让获得授权的用户以另外一个用户的身份运行指定的命令。授权文件为 `/etc/sudoers`，此文件有特定的语法格式，因此有个专用的编辑命令 `visudo`,其在退出时，可以帮助我们检查语法错误。sudoers 配置如下

### 2.1 授权机制
```
$ sudo visudo
root    ALL=(ALL)       ALL
tao     ALL=(ALL)       ALL
%wheel  ALL=(ALL)       ALL

# 总结:
那个用户 从什么地方 以谁的身份  执行什么命令
who       where=(whom)       commands
users     hosts=(runas)      commands
```

#### 授权选项格式
- users: 授权用户
    - `username`: 用户名
    - `#uid`: 用户ID(UID)
    - `%groupname`: 用户组名称
    - `%#gid`: 用户组ID(GID)
    - `user_alias`: 用户别名
- hosts: 用户登陆限制，只有在限制范围内登陆的用户才能使用授权的命令
    - `ip`: ip 地址
    - `hostname`: 域名
    - `NetAddr`: 子网
    - `host_alias`: 网络别名
- runas: 以哪些用户的身份执行命令
- commands: 授权的命令，必需是全路经
    - `command`: 命令
    - `directory`: 目录
    - `sudoedit`：特殊权限，可用于向其它用户授予sudo权限
    - `cmnd_alias`: 命令别名

#### wheel 组
wheel 组是 Linux 中的特殊组即管理员组，属于 wheel 组的成员均具有所有管理员权限

```
# root 身份执行
$ usermod pythoner -a -G wheel

$ su - pythoner    # 必需要以交互式登陆的方式切换到 pythoner 才能生效

$ id pythoner
uid=1001(pythoner) gid=1001(pythoner) 组=1001(pythoner),10(wheel)

$ sudo cat /etc/shadow
```

## 2.2 定义别名的方法
suders 支持设置别名，用于简化配置工作。别名类似于变量，可复用，可避免重复输入。别名设置的语法格式为:

`ALIAS_TYPE    NAME=item1, item2, item3, ...`
- `NAME`：别名名称，必须使用全大写字符；
- `ALIAS_TYPE`: 别名类型，分别与上面一一对应
    - `User_Alias`
    - `Host_Alias`
    - `Runas_Alias`
    - `Cmnd_Alias`: 包含的命令必需全路经

```
# 别名设置
User_Alias    NETADMIN=tom, jerry
Cmnd_Alias    NETCMND=/usr/sbin/ip, /usr/sbin/ifconfig

# 使用别名进行配置
NETADMIN    localhost=(root)    NETCMND
```

### 2.3 sudo命令s使用
`sudo    [options]    COMMAND`
- options
    - `-l`: 列出用户能执行的命令
    - `-k`: 清除此前缓存用户成功认证结果；
    - `-u`: 以哪个用户执行

默认 sudo 有检票机制，即能记录成功认证结果一段时间，默认为5分钟。`-k` 选项则可以手动取消，下此使用 sudo 时必需输入密码。同时需要提醒大家注意的是，执行 sudo 时输入的是用户自己的密码，不是 root 密码。

### 2.4 sudoers 配置示例
```
Cmnd_Alias USERADMINCMNDS = /usr/sbin/useradd, /usr/sbin/usermod, /usr/bin/passwd [a-z]*, !/usr/bin/passwd root
User_Alias USERADMIN = bob, alice
USERADMIN            ALL=(root)            NOPASSWD:USERADMINCMNDS
```            
sudoers 中常用的标签
- `NOPASSWD`:  标识使用命令时，无需输入密码
- `PASSWD`:  默认，使用命令时，需要输入密码
- `!COMMAND`: `!` 表示不允许执行 COMMAND 命令
