# 4.1 Linux目录机构
文件系统同样是很复杂的东西，具体原理后面会介绍。当前，只要知道文件系统是操作系统对磁盘的抽象，为用户提供了管理磁盘文件的接口。开机启动时，，内核加载完毕之后，内核就会挂载用户在开机启动配置文件中设置的根文件系统。Linux 上的文件系统必需挂载到根文件系统上才能被使用。开机启动流程，文件系统会在之后详细介绍。当前我们需要重点了解的是，Linux 上目录和文件的组织结构。


如同在 Windows 上创建目录和文件上一样，我们可以在Linux 上随意的创建和删除文件。但是就像我们很难在别人的Windows 系统上查找文件一样，如果各Linux 发行厂商随意的组织 Linux 的文件，当我们更换一个 Linux 发行版时，我们可能就很难找到配置文件，应用程序；程序开发者也很难统一配置程序的安装目录。所以 Linux 标准委员会为避免这种情况发生，指定了一个标准，叫 FHS(filesystem hierarchy standard)。


FHS 主要对 `/`, `/usr`, `/var` 的使用进行了规范，我们将按照这三个层次进行介绍

## 1. FHS 简介
### 1.1 官方文档简介
**This standard enables:**
- Software to predict the location of installed files and directories, and
- Users to predict the location of installed files and directories.

**We do this by:**
- Specifying guiding principles for each area of the filesystem,
- Specifying the minimum files and directories required,
- Enumerating exceptions to the principles, and
- Enumerating specific cases where there has been historical conflict.

**The FHS document is used by:**
- Independent software suppliers to create applications which are FHS compliant, and work with distributions
which are FHS complaint,
- OS creators to provide systems which are FHS compliant, and
- Users to understand and maintain the FHS compliance of a system.
The FHS document has a limited scope:
- Local placement of local files is a local issue, so FHS does not attempt to usurp system administrators.
- FHS addresses issues where file placements need to be coordinated between multiple parties such as local
sites, distributions, applications, documentation, etc.

### 1.2 FHS 标准内容概述
`/`, `/usr`, `/var` 必需包含的目录，及目录作用如下:

`/`
- `bin/`:
- `sbin/`:
- `home/`:
- `root/`:
- `lib`:
- `lib64`:
- `media/`:
- `mnt/`:
- `usr/`:
  - `bin/`:
  - `sbin/`:
  - `lib`:
  - `lib64`:
  - ``:
- `var/`
  - ``:
  - ``:
  - ``:
  - ``

## 2. / 根目录

## 3. /usr 目录

## 4. /var 目录
