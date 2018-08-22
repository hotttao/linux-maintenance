# 23.1 ftp基础入门
ftp 全称为 file transfer protocol，文件传输协议。ftp 诞生与互联网的早期，目标是完成文件传输，所以其传输数据的方式比较奇葩，本节我们就来对 ftp 做一个简单介绍。

## 1. ftp 简介
### 1.1 ftp 传输过程
![ftp](../images/22/ftp_protocol.jpg)

如上图所示，ftp 的连接分为两类
1. 命令连接：传输命令
2. 数据连接：传输数据

当需要传输数据时，客户端向 ftp 服务端的 21 端口发起连接请求建立连接，此连接主要用来传输客户端的命令。然后命令的操作不能在当前连接上传输，必需新建一条连接进行数据传输。

数据连接的创建有两种模式(从服务端的角度看)
1. 主动模式(PORT)：Server端向客户端发起连接请求，请求的端口为命令连接使用的端口向后的第一个可用端口发起连接请求
2. 被动模式(PASV): Server端打开一个随机端口，并通过命令连接告知客户端，并等待客户端连接

数据传输完成后，数据连接即断开，下此传输时在重新建立连接。

### 1.2 ftp 数据传输格式
ftp 不会使用 MIME 对数据进行编码，ftp 会自动根据要传输的数据是文本格式还是二进制格式来选择传输机制。

### 1.3 ftp 的认证机制
Linux 上有一个提供认证的共享服务 PAM(Pluggable Authenticate Module),PAM 是一个认证框架包括各种库，是高度模块化的，我们 ftp 就是调用 PAM 的服务提供认证功能的。

```
$ rpm -ql pam
/etc/pam.d
/etc/pam.d/config-util
/etc/pam.d/fingerprint-auth
/etc/pam.d/other
/etc/pam.d/password-auth
/etc/pam.d/postlogin
/etc/pam.d/smartcard-auth
/etc/pam.d/system-auth
/etc/security

# pam 的模块目录,每一个模块可以实现一种认证功能
/usr/lib64/security  
/usr/lib64/security/pam_access.so
/usr/lib64/security/pam_chroot.so
............

# 所有调用 pam 进行认证的服务如何进行认证，由此目录下的配置文件配置
/etc/pam.d     
/etc/pam.d/config-util
/etc/pam.d/fingerprint-auth
/etc/pam.d/other
/etc/pam.d/password-auth
/etc/pam.d/postlogin
......
```

### 1.4 协议实现
1. Server 端：
	- Windows: Serv-U, IIS, Filezilla
	- 开源：wuftpd, proftpd, pureftpd, vsftpd(Very Secure FTP daemon), ...
2. Client 端：
	- Windows：ftp, Filezilla, CuteFTP, FlashFXP, ...
	- 开源：lftp, ftp, Filezilla, gftp, ...

## 2. vsftpd 简介
vsftpd 全称是非常安全的 ftp 服务，以ftp用户的身份运行进程，默认认用户即为ftp用户，匿名用户的默认路径即ftp用户的家目录`/var/ftp`

```
rpm -ql vsftpd
$ rpm -ql vsftpd
/etc/logrotate.d/vsftpd
/etc/pam.d/vsftpd                      # pam 认证配置文件
/etc/vsftpd                            # 配置文件目录
/etc/vsftpd/ftpusers
/etc/vsftpd/user_list
/etc/vsftpd/vsftpd.conf                # 配置文件
/etc/vsftpd/vsftpd_conf_migrate.sh
/usr/lib/systemd/system-generators/vsftpd-generator
/usr/lib/systemd/system/vsftpd.service  # 作为独立服务
/usr/lib/systemd/system/vsftpd.target   # 作为托管服务
/usr/lib/systemd/system/vsftpd@.service
/usr/sbin/vsftpd
```

### 2.1 路经映射
ftp 也是通过 URL 进行资源定位的 `SCHEME://username:password@HOST:PORT/PATH/TO/FILE`。每个用户的URL的`/`映射到当前用户的家目录。yum 安装 vsftpd 时默认会创建 ftp 用户，匿名访问 ftp 服务时，匿名用户将自动映射为 ftp 用户。匿名用户又可称为 `anonymous`。所以匿名用户的`/` 为 ftp 用户的家目录 `/var/ftp/`。

```
$ grep "^ftp" /etc/passwd
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin

$ systemctl start vsftpd.service

# 默认就是匿名用户登陆
$ lftp 192.168.1.106
lftp 192.168.1.106:~> ls
drwxr-xr-x    2 0        0               6 Aug 03  2017 pub

# 使用 ftp 匿名登陆
$ lftp -u ftp 192.168.1.106
口令:
lftp ftp@192.168.1.106:~> ls            
drwxr-xr-x    2 0        0               6 Aug 03  2017 pub

# 使用 anonymous 匿名登陆
$ lftp -u anonymous 192.168.1.106
口令:
lftp anonymous@192.168.1.106:~> ls      
drwxr-xr-x    2 0        0               6 Aug 03  2017 pub
```

### 2.3 ftp 用户的权限
一个用户通过文件共享服务访问文件系统上的文件的生效权限为此用户在共享服务上拥有的共享权限与其在本地文件系统上拥有的权限的交集。
