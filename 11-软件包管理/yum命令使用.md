# 11.3 yum工具介绍
## 1. yum 概述
- CentOS: yum, dnf
- URL: ftp://172.16.0.1/pub/    
- YUM: yellow dog, Yellowdog Update Modifier
- yum repository: yum repo(yum 仓库)
    - 存储了众多rpm包，以及包的相关的元数据文件（放置于特定目录下：repodata）；
    - 包含 repodata 目录的路径，就是 yum 源应该指向的路径
- 文件服务器：
    - ftp://
    - http://
    - nfs://
    - file:///

### 1.1 yum客户端：
配置文件：
- /etc/yum.conf：为所有仓库提供公共配置
- /etc/yum.repos.d/*.repo：为仓库的指向提供配置
- whatis/man yum.conf

仓库指向的定义：
```
[repositoryID]
name=Some name for this repository
baseurl=url://path/to/repository/  # 可多个 url
enabled={1|0}
gpgcheck={1|0}                    # 包来源合法性检验
gpgkey=URL                        # 秘钥文件
enablegroups={1|0}                # 在此仓库上支持组
failovermethod={roundrobin|priority}  # baseurl 指向多个时，失败后的选择
    默认为：roundrobin，意为随机挑选；
cost=默认为1000                    # 开销，指定仓库优先级，越高越低
```

### 1.2 yum的repo配置文件中可用的变量：
- $releasever: 当前OS的发行版的主版本号；
- $arch: 平台；
- $basearch：基础平台；
- \$YUM0\-$YUM9\: 可自定义的变量
- http://mirrors.magedu.com/centos/$releasever/$basearch/os

## 2. yum命令
yum [options] [command] [package ...]

### 2.1 command is one of:
- install package1 [package2] [...]
- update [package1] [package2] [...]
- update-to [package1] [package2] [...]
- check-update
- upgrade [package1] [package2] [...]
- upgrade-to [package1] [package2] [...]
- distribution-synchronization [package1] [package2] [...]
- remove | erase package1 [package2] [...]
- list [...]
- info [...]
- provides | whatprovides feature1 [feature2] [...]
- clean [ packages | metadata | expire-cache | rpmdb | plugins | all ]
- makecache
- groupinstall group1 [group2] [...]
- groupupdate group1 [group2] [...]
- grouplist [hidden] [groupwildcard] [...]
- groupremove group1 [group2] [...]
- groupinfo group1 [...]
- search string1 [string2] [...]
- shell [filename]
- resolvedep dep1 [dep2] [...]
- localinstall rpmfile1 [rpmfile2] [...]
-  (maintained for legacy reasons only - use install)
- localupdate rpmfile1 [rpmfile2] [...]
-  (maintained for legacy reasons only - use update)
- reinstall package1 [package2] [...]
- downgrade package1 [package2] [...]
- deplist package1 [package2] [...]
- repolist [all|enabled|disabled]
- version [ all | installed | available | group-* | nogroups* | grouplist | groupinfo ]
- history [info|list|packages-list|packages-info|summary|addon-info|redo|undo|rollback|new|sync|stats]
- check
- help [command]

**显示仓库列表**：
- yum repolist [all|enabled|disabled]

**显示程序包**：
- yum list [all | glob_exp1] [glob_exp2] [...]
    - yum list  php*(通配符)
- yum list {available|installed|updates} [glob_exp1] [...]

**安装程序包**：
- yum install package1 [package2] [...]
- yum reinstall package1 [package2] [...]  (重新安装)

**升级程序包**：
- yum update [package1] [package2] [...]
- yum downgrade package1 [package2] [...] (降级)

**检查可用升级**：
- yum check-update

**卸载程序包**：
- yum remove | erase package1 [package2] [...]
    - 依赖被卸载的包的包也会被卸载

**查看程序包information**：
- yum info [package1]

**查看指定的特性(可以是某文件)是由哪个程序包所提供**：
- yum provides | whatprovides feature1 [feature2] [...]

**清理本地缓存**：
- yum clean [ packages | metadata | expire-cache | rpmdb | plugins | all ]

**构建缓存**：
- yum makecache

**搜索**：
- yum search string1 [string2] [...]
- 以指定的关键字搜索程序包名及summary信息；

**查看指定包所依赖的capabilities**：
- yum deplist package1 [package2] [...]

**查看yum事务历史**：
- yum history [info|list|packages-list|packages-info|summary|addon-info|redo|undo|rollback|new|sync|stats]

**安装及升级本地程序包**：
- 作用: 本地安装 rpm 包，centos6+ 版本中已被 yum install 所代替
- yum localinstall rpmfile1 [rpmfile2] [...] (maintained for legacy reasons only - use yum install)
- yum localupdate rpmfile1 [rpmfile2] [...] (maintained for legacy reasons only - use yum update)

**包组管理的相关命令**：
- yum groupinstall group1 [group2] [...]
- yum groupupdate group1 [group2] [...]
- yum grouplist [hidden] [groupwildcard] [...]
- yum groupremove group1 [group2] [...]
- yum groupinfo group1 [...]

### 2.2 yum的命令行选项：
- --nogpgcheck：禁止进行gpg check；
- -y: 自动回答为“yes”；
- -q：静默模式；
- --disablerepo=repoidglob：临时禁用此处指定的repo；
- --enablerepo=repoidglob：临时启用此处指定的repo；
- --noplugins：禁用所有插件；


## 3. yum 仓库管理
### 3.1 如何使用光盘当作本地yum仓库：
1. 挂载光盘至某目录，例如/media/cdrom
    - mount -r -t iso9660 /dev/cdrom /media/cdrom
2. 创建配置文件
```
[CentOS7]
name=
baseurl=file:////media/cdrom
gpgcheck=
enabled=
```

### 3.2 创建 yum 仓库
createrepo [options] <directory>
- 作用: 创建 yum 仓库所需的 repodata 目录
