# 10.5 软件包管理rpm命令使用

## 1. CentOS rpm
- 作用: 安装、升级、卸载、查询和校验、数据库维护
- rpm  [OPTIONS]  [PACKAGE_FILE]
    - 安装：-i, --install
    - 升级：-U, --update, -F, --freshen
    - 卸载：-e, --erase
    - 查询：-q, --query
    - 校验：-V, --verify
    - 数据库维护：--builddb, --initdb
        
### 1.1 安装：
rpm {-i|--install} [install-options] PACKAGE_FILE ...
- GENERAL OPTIONS：
    -v：verbose，详细信息
    -vv：更详细的输出
- [install-options]：
    - -h：hash marks输出进度条；每个#表示2%的进度；
    - --test：测试安装，检查并报告依赖关系及冲突消息等；
    - --nodeps：忽略依赖关系；不建议；
    - --replacepkgs：重新安装
    - --nosignature：不检查包签名信息，不检查来源合法性；
    - --nodigest：不检查包完整性信息；    
    - --noscripts: 不执行程序包脚本片段，包括以下四种类型的脚本
- 注意：rpm可以自带脚本，包括四类：
    - preinstall：安装过程开始之前运行的脚本，%pre ， --nopre
    - postinstall：安装过程完成之后运行的脚本，%post , --nopost
    - preuninstall：卸载过程真正开始执行之前运行的脚本，%preun, --nopreun 
    - postuninstall：卸载过程完成之后运行的脚本，%postun , --nopostun
                
## 1.2 升级：
rpm {-U|--upgrade} [install-options] PACKAGE_FILE ...
rpm {-F|--freshen} [install-options] PACKAGE_FILE ...
rpm -Uvh PACKAGE_FILE ...
rpm -Fvh PACKAGE_FILE ...
- -U：升级或安装；
- -F：升级，不存在旧版程序，不执行任何操作
- --oldpackage：降级；
- --force：强制升级；
- [install-options]
- 注意：
    - 不要对内核做升级操作；Linux支持多内核版本并存，因此，直接安装新版本内核；
    - 如果某原程序包的配置文件安装后曾被修改过，升级时，新版本的程序提供的同一个配置文件不会覆盖原有版本的配置文件，而是把新版本的配置文件重命名(FILENAME.rpmnew)后提供；
                    
### 1.3 卸载：
rpm {-e|--erase} [--allmatches] [--nodeps] [--noscripts] [--test] PACKAGE_NAME ...        
- --allmatches：卸载所有匹配指定名称的程序包的各版本；
- --nodeps：忽略依赖关系
- --test：测试卸载，dry run模式
            
### 1.4 查询：
rpm {-q|--query} [select-options] [query-options]
- [select-options]: 对哪类包进行查询
    - PACKAGE_NAME：查询指定的程序包是否已经安装，及其版本；
    - -a, --all：查询所有已经安装过的包；
    - -f  FILE：查询指定的文件由哪个程序包安装生成；
    - -g, --group <group>:
    - -p, --package PACKAGE_FILE：用于实现对未安装的程序包执行查询操作；
    - --whatprovides CAPABILITY：查询指定的CAPABILITY由哪个程序包提供；
    - --whatrequires CAPABILITY：查询指定的CAPABILITY被哪个包所依赖；    
- [query-options]: 查询库哪些信息
    - --changelog：查询rpm包的changlog；
    - -l, --list：程序安装生成的所有文件列表；
    - -i, --info：程序包相关的信息，版本号、大小、所属的包组，等；
    - -c, --configfiles：查询指定的程序包提供的配置文件；
    - -d, --docfiles：查询指定的程序包提供的文档；
    - --provides：列出指定的程序包提供的所有的CAPABILITY；
    - -R, --requires：查询指定的程序包的依赖关系；
    - --scripts：查看程序包自带的脚本片断；
- 用法：
    - -qi  PACKAGE, -qf FILE, -qc PACKAGE, -ql PACKAGE, -qd PACKAGE
    - -qpi  PACKAGE_FILE, -qpl PACKAGE_FILE, -qpc PACKAGE_FILE, ...

### 1.5 校验：
rpm {-V|--verify} [select-options] [verify-options]
- [select-options]: 同 query
- [verify-options]
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
        
## 2. 包来源合法性验正和完整性验正
1. 获取并导入信任的包制作者的密钥：
    - 对于CentOS发行版来说：rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7        
2. 验正：
    - 安装此组织签名的程序时，会自动执行验正；
    - 手动验正：rpm -K PACKAGE_FILE

## 3. rpm 数据库重建：
- rpm 管理器数据库路径：/var/lib/rpm/
- rpm {--initdb|--rebuilddb} [--dbpath DIRECTORY] [--root DIRECTORY]
    - --initdb：初始化数据库，当前无任何数据库可实始化创建一个新的；当前有时不执行任何操作；
    - --rebuilddb：重新构建，通过读取当前系统上所有已经安装过的程序包进行重新创建；无论当前是否存在，都会重新创建数据库
- 获取帮助：
    - CentOS 6：man rpm
    - CentOS 7：man rpmdb
    
