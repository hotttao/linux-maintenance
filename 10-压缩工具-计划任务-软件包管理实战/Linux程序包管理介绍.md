# 10.3 Linux程序包管理介绍
本节是 Linux 包管里器的一些背景知识，目的是让大家对为什么会存在包管里器，包管理器本身有个大体上的了解。在这之后我们会详细介绍 Centos 的包管理器 rpm 的使用。本节主要包含以下内容:
1. 为什么会有包管里器
2. 包管理器简介
  - 包管理器的种类
  - 包的命令格式
  - 包依赖关系的解决
  - 包的可能来源

## 1. 为什么会有包管里器


## 2. 程序包管理器
### 2.1 程序包管里器的种类
不同的主流 Linux 发行版为自家开发了特有的包管里器，目前比较流行的有如下几个:
1. debian：dpt(dpkg), 后缀名为 `.deb`
2. redhat：rpm(redhat package manager/rpm is package manager),后缀名为 `.rpm`
3. S.u.S.E：rpm, ".rpm"
4. Gentoo：ports
5. ArchLinux：dnf

### 2.1 包命名格式：
1. 源代码：
    - name-VERSION.tar.gz
    - VERSION：major.minor.release
2. rpm
    - name-VERSION-ARCH.rpm
    - VERSION：major.minor.release
    - ARCH:release.os.arch - rpm包的发行号
        - 2.el7.i386.rpm
        - archetecture：i386, x64(amd64), ppc, noarch
    - eg: redis-3.0.2.targz --> redis-3.0.2-1.centos7.x64.rpm
        - VERSION: 3.0.2
        - ARCH: 1.centos7.x64    
3. 拆包：主包和支包
    - 主包：name-VERSION-ARCH.rpm
    - 支包：name-function-VERSION-ARCH.rpm
        - function：devel, utils, libs, ...

### 2.2 依赖关系：
前端工具：自动解决依赖关系；
- yum：rhel系列系统上rpm包管理器的前端工具；
- apt-get (apt-cache)：deb包管理器的前端工具；
- zypper：suse的rpm管理器前端工具；
- dnf：Fedora 22+系统上rpm包管理器的前端工具；

**ldd**  /path/binary_file:
- 作用: 查看二进制文件依赖的库文件

**ldconfig**
- 作用: 管理和查看本机的挂载库文件
- -p: 显示本机已经缓存的所有可用库文件及文件路径映射关系
- 配置文件: `/etc/ld.so.conf`, `/etc/ld.so.conf.d/*.conf`
- 缓存文件: /etc/ld.so.cache


### 2.3 程序包管理器
- 功能：将编译好的应用程序的各组成文件打包成一个或几个程序包文件，从而更方便地实现程序包的安装、升级、卸载和查询等管理操作；

#### 程序包组成
1. 程序包的组成清单（每个程序包都单独 实现）；
    - 文件清单
    - 安装或卸载时运行的脚本
2. 数据库（公共）
    - 程序包的名称和版本；
    - 依赖关系；
    - 功能说明；
    - 安装生成的各文件的文件路径及校验码信息；
    - 等等等
    - /var/lib/rpm/

#### 获取程序包的途径:
- 系统发行版的光盘或官方的文件服务器（或镜像站点）：
    - http://mirrors.aliyun.com,
    - http://mirrors.sohu.com,
    - http://mirrors.163.com
- 项目的官方站点
- 第三方组织：
    - EPEL
    - 搜索引擎
        - http://pkgs.org
        - http://rpmfind.net
        - http://rpm.pbone.net
- 自动动手，丰衣足食
- 建议：
    - 检查其合法性
    - 来源合法性；
    - 程序包的完整性；
