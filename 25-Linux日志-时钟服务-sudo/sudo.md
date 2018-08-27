# 25.3 sudo应用与实战

## 1. su：switch user
用户切换
- su    -l  user  == su -  user
- su    -l  user    -c  'COMMAND'

## 2. sudo：
作用: 能够让获得授权的用户以另外一个用户的身份运行指定的命令；
### 2.1 授权机制：
**授权文件**
- /etc/sudoers
- 编译此文件的专用命令：visudo
**授权项**：
`who      where=(whom)       commands`
`users    hosts=(runas)        commands`
- eg:
    - root        ALL=(ALL)            ALL
    - %wheel    ALL=(ALL)            ALL
- users：
    - username
    - \#uid
    - %groupname
    - %#gid
    - user_alias 支持将多个用户定义为一组用户，称之为用户别名，即user_alias；
- hosts:
    - ip
    - hostname
    - NetAddr
    - host_alias
- runas:
    - runas_alias(user_alias)
- commands:
    - command
    - directory
    - sudoedit：特殊权限，可用于向其它用户授予sudo权限；
    - cmnd_alias

## 2.2 定义别名的方法：
ALIAS_TYPE    NAME=item1, item2, item3, ...
- NAME：别名名称，必须使用全大写字符；
- ALIAS_TYPE:
    - User_Alias
    - Host_Alias
    - Runas_Alias
    - Cmnd_Alias
- 例如：
```
# 别名设置
User_Alias    NETADMIN=tom, jerry
Cmnd_Alias  NETCMND=ip, ifconfig, route
# 使用别名进行配置
NETADMIN    localhost=(root)    NETCMND
```

### 2.3 sudo命令：
sudo    [options]    COMMAND
- 检票机制：能记录成功认证结果一段时间，默认为5分钟；
- options
    - -l: 列出用户能执行的命令
    - -k: 清除此前缓存用户成功认证结果；
    - -u: 以哪个用户执行
### 2.4  /etc/sudoers应用示例：
```
Cmnd_Alias USERADMINCMNDS = /usr/sbin/useradd, /usr/sbin/usermod, /usr/bin/passwd [a-z]*, !/usr/bin/passwd root
User_Alias USERADMIN = bob, alice
USERADMIN            ALL=(root)            NOPASSWD:USERADMINCMNDS
```            
常用标签：
- NOPASSWD:  标识使用命令时，无需输入密码
- PASSWD:  默认，使用命令时，需要输入密码
