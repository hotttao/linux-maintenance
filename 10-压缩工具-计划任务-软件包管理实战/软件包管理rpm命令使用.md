# 10.5 软件包管理rpm命令使用
本节我们主要来讲解 rpm 命令的使用。rpm 可实现程序的安装、卸载和升级。但相比于程序的管理，rpm 的查询命令能帮助我们快速找到文件或二进制程序所属的程序包，及程序包的配置文件等信息，反而更加重要。由于程序之间存在依赖关系，而 rpm 不能自动帮我们解决程序的依赖问题，因此在程序的管理更加常用的命令是 rpm 的前端管理工具 yum。yum 能自动帮我们解决程序的依赖问题，我们会在下个章节介绍 yum 的使用。

## 1. CentOS rpm
rpm 提供了应用程序的安装、升级、卸载、查询、校验和数据库维护，其使用方式如下。我们会分段讲解各个命令的使用

`rpm  [OPTIONS]  [PACKAGE_FILE]`
- 子命令选项:
    - 安装：`-i, --install`
    - 升级：`-U, --update, -F, --freshen`
    - 卸载：`-e, --erase`
    - 查询：`-q, --query`
    - 校验：`-V, --verify`
    - 数据库维护：`--builddb, --initdb`
- 通用选项:
    `-v`：verbose，详细信息
    `-vv`：更详细的输出

### 1.1 安装
`rpm {-i|--install} [install-options] PACKAGE_FILE ...`
- `[install-options]`：
    - `-h`：hash marks输出进度条；每个#表示2%的进度；
    - `--test`：测试安装，检查并报告依赖关系及冲突消息等；
    - `--nodeps`：忽略依赖关系；不建议；
    - `--replacepkgs`：重新安装
    - `--nosignature`：不检查包签名信息，不检查来源合法性；
    - `--nodigest`：不检查包完整性信息；    
    - `--noscripts`: 不执行程序包脚本片段，包括以下四种类型的脚本
- 注意：rpm可以自带脚本，包括四类：
    - `preinstall`：安装过程开始之前运行的脚本，%pre ， `--nopre` 可禁止执行此类脚本
    - `postinstall`：安装过程完成之后运行的脚本，%post , `--nopost` 可禁止执行此类脚本
    - `preuninstall`：卸载过程真正开始执行之前运行的脚本，%preun, `--nopreun` 可禁止执行此类脚本
    - `postuninstall`：卸载过程完成之后运行的脚本，%postun , `--nopostun` 可禁止执行此类脚本

## 1.2 升级：
`rpm {-U|--upgrade} [install-options] PACKAGE_FILE ...`  
`rpm {-F|--freshen} [install-options] PACKAGE_FILE ...`  
`rpm -Uvh PACKAGE_FILE ...`    
`rpm -Fvh PACKAGE_FILE ...`  
- `-U`：升级或安装；
- `-F`：升级，不存在旧版程序，不执行任何操作
- `--oldpackage`：降级；
- `--force`：强制升级；
- `[install-options]`: 所有安装时可用选项，升级亦可用
- 注意：
    - 不要对内核做升级操作；Linux支持多内核版本并存，因此，直接安装新版本内核；
    - 如果某原程序包的配置文件安装后曾被修改过，升级时，新版本的程序提供的同一个配置文件不会覆盖原有版本的配置文件，而是把新版本的配置文件重命名(FILENAME.rpmnew)后提供；

### 1.3 卸载：
`rpm {-e|--erase} [--allmatches] [--nodeps] [--noscripts] [--test] PACKAGE_NAME ...`        
- `--allmatches`：卸载所有匹配指定名称的程序包的各版本；
- `--nodeps`：忽略依赖关系
- `--test`：测试卸载，dry run模式

### 1.4 查询：
`rpm {-q|--query} [select-options] [query-options]`
- `[select-options]`: 通过什么查询程序包
    - `PACKAGE_NAME`：查询指定的程序包是否已经安装，及其版本；
    - `-a, --all`：查询所有已经安装过的包；
    - `-f  FILE`：查询指定的文件由哪个程序包安装生成；
    - `-g, --group <group>`:
    - `-p, --package PACKAGE_FILE`：用于实现对未安装的程序包执行查询操作；
    - `--whatprovides CAPABILITY`：查询指定的CAPABILITY由哪个程序包提供；
    - `--whatrequires CAPABILITY`：查询指定的CAPABILITY被哪个包所依赖；    
- `[query-options]`: 查询包的哪些信息
    - `--changelog`：查询rpm包的changlog；
    - `-l, --list`：程序安装生成的所有文件列表；
    - `-i, --info`：程序包相关的信息，版本号、大小、所属的包组，等；
    - `-c, --configfiles`：查询指定的程序包提供的配置文件；
    - `-d, --docfiles`：查询指定的程序包提供的文档；
    - `--provides`：列出指定的程序包提供的所有的CAPABILITY；
    - `-R, --requires`：查询指定的程序包的依赖关系；
    - `--scripts`：查看程序包自带的脚本片断；
- 用法：
    - 查询已安装包: `-qi  PACKAGE`, `-qf FILE`, `-qc PACKAGE`, `-ql PACKAGE`, `-qd PACKAGE`
    - 查询未安装包: `-qpi  PACKAGE_FILE`, `-qpl PACKAGE_FILE`, `-qpc PACKAGE_FILE`, ...


### 1.5 校验：
`rpm {-V|--verify} [select-options] [verify-options]`
- `[select-options]`: 同 query
- `[verify-options]`

```
> rpm -V zsh


- 检验结果:                
    - S file Size differs
    - M Mode differs (includes permissions and file type)
    - 5 digest (formerly MD5 sum) differs
    - D Device major/minor number mismatch
    - L readLink(2) path mismatch
    - U User ownership differs
    - G Group ownership differs
    - T mTime differs
    - P caPabilities differ
```

## 2. 包来源合法性验正和完整性验正
网络数据的合法性和完整性验证需要使用到加密技术，在后续的 web 服务章节我们会详细讲解加密算法在数据合法性和完整性上的应用，此处我们简单介绍一下。

加密算法分为对称加密，单向加密和非对称加密三类。
1. 对称加密指的是加密和解密使用的是同一种密钥
2. 单向加密，只要源数据发生微弱变化，加密结果会发生巨大变化，它通常用于提取指纹信息，被用作完整性验证
3. 非对称加密，有公钥和私钥两部分组成，使用私钥加密的数据只能使用公钥解密，反之亦然。如果某人用其私钥加密了某个数据，我们用其私钥能够解密就可以说明数据来自他。

因此在包来源合法性和完整性验证过程中，包的制作者首先使用单向加密获取包的指纹信息，并将其用自己的私钥加密制作成数据签名。由于其公钥所有人都可以获取，包下载者下载包后，使用其公钥解密数字签名，如果能够解密说明包的确来自包制作者。然后在使用同样的单向加密算法，对包进行加密，将加密结果与数字签名内的指纹信息进行比对，如果相同说明包是完整的。

使用 rpm 验证包来源合法性和完整性包括如下两个步骤:
1. 获取并导入信任的包制作者的公钥：
	- `rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7`
    - 对于CentOS发行版来说公钥位于 `/etc/pki/rpm-gpg` 目录内        
2. 验正：
    - 安装此组织签名的程序时，会自动执行验正；
    - 手动验正：`rpm -K PACKAGE_FILE`

## 3. rpm 数据库重建
rpm 数据库记录了所有包的基本信息，所属文件，及其所属文件的存放路径等信息。rpm 查询操作都是基于此数据库进行的。rpm 管理器数据库在`/var/lib/rpm/` 下。如果数据库出现损坏，可使用 rpm 命令进行修复，修复命令如下

`rpm {--initdb|--rebuilddb} [--dbpath DIRECTORY] [--root DIRECTORY]`
- `--initdb`：初始化数据库，当前无任何数据库可实始化创建一个新的；当前存在数据库时不执行任何操作；
- `--rebuilddb`：重新构建，通过读取当前系统上所有已经安装过的程序包进行重新创建；无论当前是否存在，都会重新创建数据库
- 获取帮助：
    - CentOS 6：`man rpm`
    - CentOS 7：`man rpmdb`
