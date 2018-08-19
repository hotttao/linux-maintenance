# 20.8 httpd 辅助工具
本章的最后一节我们来学习 httpd 提供了辅助工具的使用，包括:
1. httpd: httpd 服务的主程序
2. apachectl：httpd自带的服务控制脚本，支持start和stop,restart；
1. htpasswd：basic认证基于文件实现时，用到的账号密码文件生成工具；
3. apxs：由httpd-devel包提供，扩展httpd使用第三方模块的工具；
4. rotatelogs：日志滚动工具；
5. suexec：访问某些有特殊权限配置的资源时，临时切换至指定用户身份运行；
6. ab： apache benchmark

## 1. httpd
`httpd[.event|worker] OPTIONS`
- 作用: httpd 主程序
- 选项:
	- `-t`: 仅对配置文件执行语法检查。程序在语法解析检查结束后立即退出，或者返回"0"(OK)，或者返回非0的值(Error)。如果还指定了"-D DUMP_VHOSTS"，则会显示虚拟主机配置的详细信息
	- `-l`: 输出一个静态编译在服务器中的模块的列表。它不会列出使用LoadModule指令动态加载的模块。
	- `-L`: 输出一个指令的列表，并包含了各指令的有效参数和使用区域。
	- `-M`: 输出一个已经启用的模块列表，包括静态编译在服务器中的模块和作为DSO动态加载的模块。
	- `-v`: 显示httpd的版本，然后退出。
	- `-V`: 显示httpd和APR/APR-Util的版本和编译参数，然后退出。
	- `-X`: 以调试模式运行httpd 。仅启动一个工作进程，并且服务器不与控制台脱离
	- `-d serverroot`: 将ServerRoot指令设置初始值为serverroot。它可以被配置文件中的ServerRoot指令所覆盖。
	- `-f config`: 在启动中使用config作为配置文件。如果config不以"/"开头，则它是相对于ServerRoot的路径
	- `-k start|restart|graceful|stop|graceful-stop`: 发送信号使httpd启动、重新启动或停止 。
	- `-C directive`: 在读取配置文件之前，先处理directive的配置指令。
	- `-c directive`: 在读取配置文件之后，再处理directive的配置指令。
	- `-D parameter`: 设置参数parameter ，它配合配置文件中的<IfDefine>段，用于在服务器启动和重新启动时，有条件地跳过或处理某些命
	- `-e level`: 在服务器启动时，设置LogLevel为level 。它用于在启动时，临时增加出错信息的详细程度，以帮助排错。
	- `-E file`: 将服务器启动过程中的出错信息发送到文件file 。
	- `-R directory`: 当在服务器编译中使用了SHARED_CORE规则时，它指定共享目标文件的目录为directory 。
	- `-h`: 输出一个可用的命令行选项的简要说明。
	- `-S`: 显示从配置文件中读取并解析的设置结果(目前仅显示虚拟主机的设置)
	- `-T`: 在启动/重启的时候跳过根文件检查 (该参数在Apache 2.2.17及其以后版本有效)
- `-t` 选项的扩展:
	- `httpd -t -D DUMP_VHOSTS` : 显示虚拟主机的配置
	- `httpd -t -D DUMP_RUN_CFG` : show parsed run setting
	- `httpd -t -D DUMP_MODULES` : 显示所有已经启动的模块
	- `httpd -M` : `httpd -t -D DUMP_MODULES` 的快捷方式

```
$ httpd -l
Compiled in modules:
  core.c
  mod_so.c
  http_core.c

$ httpd -M
Loaded Modules:
 core_module (static)
 so_module (static)
 http_module (static)
 access_compat_module (shared)
 actions_module (shared)
 .......

$ httpd -t
Syntax OK
```

## 2. apachectl
`apachectl OPTIONS`
- 作用: 是slackware内附Apache HTTP服务器的script文件，可供管理员控制服务器
- 选项:
	- `configtest`: 检查设置文件中的语法是否正确。
	- `fullstatus`: 显示服务器完整的状态信息。
	- `graceful`: 重新启动Apache服务器，但不会中断原有的连接。
	- `help`: 显示帮助信息。
	- `restart`: 重新启动Apache服务器。
	- `start`: 启动Apache服务器。
	- `status`: 显示服务器摘要的状态信息。
	- `stop`: 停止Apache服务器
- 说明: `httpd` 命令的所有选项， `apachectl` 均可用

## 3. htpasswd
`htpasswd OPTIONS passwordfile username [password]`
- 作用: 用于创建和更新储存用户名、域和用户基本认证的密码文件
- 参数:
	- `passwordfile`: 密码文件的路经，使用 `-n` 选项时，无需此参数
	- `username`: 用户名
	- `password`: 密码，使用`-b` 选项时必需，默认显示提示符让用户输入密码
- 选项:
	- `-c`：创建一个加密文件，文件已经存在会删除重建
	- `-b`：在命令行中一并输入用户名和密码而不是根据提示输入密码
	- `-D`：删除指定的用户
	- `-n`：不更新加密文件，只将加密后的用户名密码显示在屏幕上
	- `-m`：默认采用MD5算法对密码进行加密
	- `-d`：采用CRYPT算法对密码进行加密
	- `-p`：不对密码进行进行加密，即明文密码
	- `-s`：采用SHA算法对密码进行加密

```
$ htpasswd -c /tmp/.httpd tao  # 首次创建文件，需要使用 -c
New password:
Re-type new password:
Adding password for user tao

$ htpasswd -b /tmp/.httpd pythoner python # 非首次创建不能使用 `-c` 否则会删除已有文件
Adding password for user pythoner
```

## 3. curl
`curl  [options]  [URL...]`
- 作用:
    - curl是基于URL语法在命令行方式下工作的文件传输工具
    - 支持FTP, FTPS, HTTP, HTTPS, GOPHER, TELNET, DICT, FILE及LDAP等协议
    - 支持HTTPS认证，并且支持HTTP的POST、PUT等方法， FTP上传， kerberos认证，HTTP上传，代理服务器， cookies， 用户名/密码认证， 下载文件断点续传，上载文件断点续传, http代理服务器管道（ proxy tunneling）， 甚至它还支持IPv6， socks5代理服务器,，通过http代理服务器上传文件到FTP服务器等等，功能十分强大。
- options:
    - `-e/--referer <URL>`:  来源网址
    - `-A/--user-agent <string>`:  设置用户代理发送给服务器
    - `-H/--header <line>`: 自定义首部信息传递给服务器
    - `-I/--head` 只显示响应报文首部信息
    - `--basic`: 使用HTTP基本认证
    - `-u/--user <user[:password]>`: 设置服务器的用户和密码
    - `--cacert <file>`:  CA证书 (SSL)
    - `--compressed` 要求返回是压缩的格式
    - `--limit-rate <rate>`:  设置传输速度
    - `-0/--http1.0`: 使用HTTP 1.0
    - `--tcp-nodelay`: 使用TCP_NODELAY选项
```

```

## 5. elinks   
`elinks  [OPTION]... [URL]...`
- 作用: 文本浏览器
- 选项:
	- `-dump`: 不进入交互式模式，而直接将URL的内容输出至标准输出；

## 6. httpd的压力测试工具
市面上常见的 web 压力测试工具有以下几种:
- 命令行工具: `ab`, `webbench`, `http_load`, `seige`
- 图形化工具: `jmeter`, `loadrunner`
- 模拟真实请求: `tcpcopy`，网易开发，复制生产环境中的真实请求，并将之保存下来；

`ab  [OPTIONS]  URL`
- 全称: apache benchmark
- 选项:
	- `-n`：总请求数；
	- `-c`：模拟的并行数；
	- `-k`：以持久连接模式 测试；
- 附注: `ulimit -n num ` 调整当前用户能同时打开的文件数
