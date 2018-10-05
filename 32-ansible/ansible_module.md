# 32.6 ansible 常用模块
上一节我们对 ansible 做了一个概括性的介绍，本节我们来看看 ansible 主程序与常见模块的使用，模块是我们定义服务配置的关键。

## 1. ansible 核心程序
ansible 的核心程序有三个
1. ansible: ad-hoc 执行命令
2. ansible-doc: ansible 插件(模块)文档查看工具
3. ansible-playbook: playbook 执行命令

### 1，1 ansible
`ansible <host-pattern> [-m module_name] [-a args] options`
- 作用: ansible 命令行工具
- 模块:
  - `[-m module_name]`: 指定使用的模块
  - `[-a args]`: 传递给模块的参数
- 选项: ansible 命令行工具的选项可分为三类
  1. 通用选项
  2. 连接选项
  3. 权限选项
- 通用选项:
  - `-C, --check`: 不实际执行，只显示程序执行可能的结果
  - `-D, --diff`: 当执行的命令改变了文件或模板的内容时，显示更改前后的内容比较，最好和 `-C, --check` 一起使用
  - `-e EXTRA_VARS, --extra-vars=EXTRA_VARS`: 向 ansible 传递的额外参数，参数值必需是行如`key=value`的键值对
  - `-f FORKS, --forks=FORKS`: 并发操作的最大机器数
  - `-i INVENTORY, --inventory=INVENTORY, --inventory-file=INVENTORY`: 定义 inventory 文件位置
  - `--list-hosts`: 只显示被操作的主机
  - `--syntax-check `:只对 playbook 执行语法检查, 不执行
  - `-t TREE, --tree=TREE`: 日志的输出目录
  - `--version`: 显示 ansible 的版本信息
- 连接选项:
  - `--private-key=PRIVATE_KEY_FILE`: 指定连接的密钥文件
  - `--key-file=PRIVATE_KEY_FILE`:指定连接的密钥文件
  - `-u,--user=REMOTE_USER`: 连接到被管控主机的帐户，默认为 None
  - `-c, --connection=CONNECTION`: 连接的类型，默认为 smart
  - `-T, --timeout=TIMEOUT`: 连接超时时长
- 权限选项:
  - `-b, --become`:
  - `--become-user=BECOME_USER`: 提权限操作切换到的用户，默认为 root
  - `--become-method=BECOME_METHOD`: 进行权限升级时使用的操作，默认为 sudo，可选值包括`sudo | su`
  - `--ask-become-pass`: 使用 sudo 或 su 时，使用的密码

#### host-pattern
ansible 支持多种主机匹配方式，以便我们能灵活的控制要操作的主机范围。常见的方式有如下几种

```bash
# 1. 全部主机
all
*

# 2. IP地址或系列主机名
one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*

# 3. 一个或多个groups
webservers             # 单个组
webservers:dbservers   # 多个组的并集
webservers:&staging    # 多个组的交集
webservers:!phoenix    # ! 表示排除关系，隶属 webservers 组但同时不在 phoenix组

# 4. host names, IPs , groups都支持通配符
*.example.com
*.com

# 5. 通配和groups的混合使用
one*.com:dbservers

# 6. 应用正则表达式，只需要以 ‘~’ 开头
~(web|db).*\.example\.com

# 7. 通过 --limit 标记来添加排除条件
ansible-playbook site.yml --limit datacenter2

# 8. 从文件读取hosts,文件名以@为前缀即可
ansible-playbook site.yml --limit @retry_hosts.txt
```

### 1.2 ansible-doc
`ansible-doc options`
- 作用: ansible 文档查看工具
- 选项:
  - `-l, --list`: 显示所有可用插件及模块
  - `-s, --snippet=module`: 显示指定插件的的帮助信息
  - `-t, --type=TYPE`: 指定被选择的插件类型，默认为 module

```
~$ ansible-doc -s shell
- name: Execute commands in nodes.
  shell:
      chdir:                 # cd into this directory before running the command
      creates:               # a filename, when it already exists, this step will *not* be run.
      executable:            # change the shell used to execute the command. Should be an absolute path to the executable.
      free_form:             # (required) The shell module takes a free form command to run, as a string.  There's not an actual option named
                               "free form".  See the examples!
      removes:               # a filename, when it does not exist, this step will *not* be run.
      stdin:                 # Set the stdin of the command directly to the specified value.
      warn:                  # if command warnings are on in ansible.cfg, do not warn about this particular line if set to no/false.
```

ansible-doc 显示的参数都是可以在 `ansible` 命令中 通过 `-a` 选项中传递给模块的参数

```
ansible -m shell -a "echo 'test' chdir=/root"
```

### ansible-playbook
`ansible-playbook [options] playbook.yml [playbook2 ...]`
- 作用: playbook 的执行命令
- 参数: `playbook.yml...` 表示 playbook 的路经
- 选项: `ansible-playbook` 与 `ansible` 命令行工具的选项基本类似
  - ` --playbook-dir=BASEDIR`: playbook 的根目录，这个根目录的设置会影响 `roles/ group_vars/` 等目录的查找路经
  - ` -t TAGS, --tags=TAGS`: 运行指定标签对应的任务

## 2.ansible 常用模块
ansible 命令的执行是为了达到期望的状态，如果被管控主机的当前状态与命令指定的状态不一致，则执行命令，所以ansible 的命令都是通过 state 参数指定要进行的操作。

### 2.1 基本模块
#### user
`ansible -m user -a "options"`
- 作用: 用户管理
- 选项:
  - `name`: 用户名
  - `uid`: 指定创建用户的 uid
  - `shell`: 设置默认shell
  - `group`: 设置默认组
  - `groups`: 设置附加组，默认操作是替换
  - `append`: `groups` 操作为追加而不是替换
  - `home`: 家目录
  - `system`: yes|no 是否为系统用户
  - `move`: 更新用户家目录时，是否将原有家目录的内容移动到新的目录中去
  - `state`: 用户操作
    - `present`: 创建用户
    - `absent`: 删除用户

```
$ ansible-doc -s user
$ ansible all -m user -a "name=test uid=3000 shell=/bin/tsh groups=testgrp"
```

#### group
`ansible -m user -a "options"`
- 作用: 用户组管理
- 选项:
  - `name`: 组名
  - `gid`: 指定gid
  - `system`: yes|no 是否为系统用户
  - `state`: 目标状态 `present|absent`


#### copy
`ansible -m copy -a 'options'`
- 作用: 文件复制和创建
- 选项:
  - `backup`：在覆盖之前将原文件备份，备份文件包含时间信息。有两个选项：yes|no
  - `content`：用于替代"src",可以直接设定指定文件的值
  - `dest`：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录
  - `directory_mode`：递归的设定目录的权限，默认为系统默认权限
  - `force`：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes
  - `others`：所有的file模块里的选项都可以在这里使用
  - `src`：要复制到远程主机的文件在本地的地址，可以是绝对路径，也可以是相对路径。如果路径是一个目录，它将递归复制。在这种情况下，如果路径使用"/"来结尾，则只复制目录里的内容，如果没有使用"/"来结尾，则包含目录在内的整个内容全部复制，类似于rsync
  - `remote_src`: yes|no，指定 scr 参数的源是本机还是远程的被管理主机，no 为本机
  - `owner`: 设置目标文件的属主
  - `group`: 设置目标文件的属组
  - `mode`: 设置目标文件的权限

```
ansible -m copy -a "src=/etc/fstab dest=/tmp/fstab.ansible"
ansible -m copy -a "content='hi ansible\n' dest=/tmp/fstab.ansible mode=600"
```

#### file
`ansible -m file -a "options"`
- 作用: 文件属性管理
- 选项:
  - `force`：yes|no 是否强制创建软连接
    - 一种是源文件不存在但之后会建立的情况下，强制创建
    - 另一种是目标软链接已存在,需要先取消之前的软链，然后创建新的软链
  - `group`：定义文件/目录的属组
  - `mode`：定义文件/目录的权限
  - `owner`：定义文件/目录的属主
  - `path`：必选项，定义文件/目录的路径
  - `recurse`：递归的设置文件的属性，只对目录有效
  - `src`：要被链接的源文件的路径，只应用于`state=link`的情况
  - `dest`：被链接到的路径，只应用于state=link的情况
  - `state`：  
    - `directory`：如果目录不存在，创建目录
    - `file`：即使文件不存在，也不会被创建
    - `link`：创建软链接
    - `hard`：创建硬链接
    - `touch`：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
    - `absent`：删除目录、文件或者取消链接文件

```
ansible test -m file -a "src=/etc/fstab dest=/tmp/fstab state=link"
ansible test -m file -a "path=/tmp/fstab state=absent"
ansible test -m file -a "path=/tmp/test state=touch"
```

#### template
`ansible -m template -a 'option'`
- 作用: 基于 python jinja2 模板生成文件并复制到目标主机
- 选项:
  - `backup`：在覆盖之前将原文件备份，备份文件包含时间信息。有两个选项：yes|no
  - `src`：要被链接的源文件的路径，只应用于`state=link`的情况
  - `dest`：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录
  - `force`：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes
  - `owner`: 设置目标文件的属主
  - `group`: 设置目标文件的属组
  - `mode`: 设置目标文件的权限

### 2.2  命令执行
#### command
`ansible -m command -a 'option'`
- 作用: 命令执行，但是无法解析 bash 中的特殊字符，比如 `|`，只能执行简单命令
- 选项:
  - `free_form`: 非参数名称，指代任何可执行命令
  - `creates`: 文件名，2.0 后支持通配符，表示指定的文件存在时不执行命令
  - `removes`: 与 `creates` 相反，表示文件不存在时不执行命令
  - `chdir`: 指定命令运行的当前目录

```
ansible -m command -a "ifconfig"
```

#### shell
`ansible -m shell -a 'option'`
- 作用: 命令执行，能正常解析 shell 语法
- 选项:
  - `free_form`: 非参数名称，指代任何可执行命令
  - `creates`: 文件名，2.0 后支持通配符，表示指定的文件存在时不执行命令
  - `removes`: 与 `creates` 相反，表示文件不存在时不执行命令
  - `chdir`: 指定命令运行的当前目录
  - `executable`: 执行运行命令的 shell 解释器，必需是绝对路经

```
ansible -m command -a "echo pswd|password --stdin tao"
```

#### script
`ansible -m script -a 'option'`
- 作用: 将脚本复制到管控主机并执行
- 选项:
  - `free_form`: 非参数名称，指代任何可执行命令
  - `creates`: 文件名，2.0 后支持通配符，表示指定的文件存在时不执行命令
  - `removes`: 与 `creates` 相反，表示文件不存在时不执行命令
  - `chdir`: 指定命令运行的当前目录
  - `executable`: 执行运行命令的 shell 解释器，必需是绝对路经

```
ansible -m script -a "mount.sh"
```

#### ping
`ansible -m ping -a 'option'`
- 作用: 测试主机是否是通的
- 选项：无

```
ansible 10.212.52.252 -m ping
10.212.52.252 | success >> {
    "changed": false,
    "ping": "pong"
}
```

### 2.3 程序安装
#### yum
`ansible -m yum -a 'option'`
- 作用: 文件属性管理
- 选项：
  - `config_file`：yum的配置文件
  - `disable_gpg_check`：关闭gpg_check
  - `disablerepo`：不启用某个源
  - `enablerepo`：启用某个源
  - `name`：要进行操作的软件包的名字，可附带版本信息，也可以传递一个url或者一个本地的rpm包的路径
  - `allow_downgrade`: 是否允许降级安装，默认为 no；默认的安装操作相当于 `yum -y update`，如果 `name` 指定的版本相对于已安装的版本较低，则不会安装
  - `state`：状态（present，absent，latest）

```
ansible test -m yum -a 'name=httpd state=latest'
ansible test -m yum -a 'name="@Development tools" state=present'
ansible test -m yum -a 'name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present'
```

#### pip
`ansible -m pip -a 'option'`
- 作用: 文件属性管理
- 选项：
  - `chdir`: pip 命令运行前切换到此目录
  - `executable`: 指定运行 pip的版本，pip 的名称或绝对路经；不能与`virtualenv`同时使用
  - `extra_args`: 传给 `pip`的额外参数
  - `name`: 安装的程序包名称，可以是一个 url
  - `version`: 指定的Python库的安装版本
  - `virtualenv`：virtualenv 虚拟环境目录，不能与 `executable` 同时使用，如果虚拟环境不存在，将自动创建
  - `virtualenv_command`: 虚拟环境使用的管理命令或绝对路经，eg:`pyvenv, ~/bin/virtualenv`
  - `virtualenv_python`: 虚拟环境中的 python 版本，当virtualenv_command使用pyvenv或-m venv模块时，不应使用此参数
  - `state`:
    - `present`:默认的，表示为安装
    - `lastest`: 安装为最新的版本
    - `absent`：表示删除
    - `forcereinstall`：“forcereinstall”选项仅适用于可ansible 2.1及更高版本

```
# 支持 pipenv 么？
ansible -m pip -a "name=ipython virtualenv=/opt/vdd/project virtualenv_command=pipenv"
```

### 2.4 服务管理
#### cron
`ansible -m cron -a 'option'`
- 作用: 周期性任务管理
- 选项：
  - `backup`：对远程主机上的原任务计划内容修改之前做备份
  - `cron_file`：如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户的任务计划
  - `day`：日
  - `hour`：小时
  - `minute`：分钟
  - `month`：月
  - `weekday`：周
  - `job`：要执行的任务，依赖于`state=present`
  - `name`：该任务的描述
  - `special_time`：指定什么时候执行，参数包括 `reboot,yearly,annually,monthly,weekly,daily,hourly`
  - `state`：确认该任务计划是创建还是删除，present or absent
  - `user`：以哪个用户的身份执行

```
ansible test -m cron -a 'name="a job for reboot" special_time=reboot job="/some/job.sh"'
ansible test -m cron -a 'name="yum autoupdate" weekday="2" minute=0 hour=12 user="root
ansible 10.212.52.252 -m cron  -a 'backup="True" name="test" minute="0" hour="2" job="ls -alh > /dev/null"'
ansilbe test -m cron -a 'cron_file=ansible_yum-autoupdate state=absent'
```

#### service
`ansible -m service -a 'option'`
- 作用: 管理服务管理
- 选项：
  - `arguments`：给命令行提供一些选项
  - `enabled`：是否开机启动 yes|no
  - `name`：必选项，服务名称
  - `pattern`：定义一个模式，如果通过status指令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务依然在运行
  - `runlevel`：运行级别
  - `sleep`：如果执行了restarted，在则stop和start之间沉睡几秒钟
  - `state`：对当前服务执行启动，停止、重启、重新加载等操作（started,stopped,restarted,reloaded）

```
ansible -m service -a "name=httpd state=started enabled=yes"
```

### 2.5 变量获取
#### setup
`ansible -m setup -a 'option'`
- 作用: 获取被管控主机的所有系统参数信息
- 选项：
  - `filter`: 参数过滤，支持 shell 通配语法
  - `gather_subset`: 限制返回的参数范围，可选值包括 `all, min, hardware, network, virtual, ohai`,值前的 `!` 表示取反
  - `gather_timeout`: 参数收集的超时时长

```
ansible 10.212.52.252 -m setup -a 'filter=ansible_*_mb'   //查看主机内存信息
ansible 10.212.52.252 -m setup -a 'filter=ansible_eth[0-2]'   //查看地接口为eth0-2的网卡信息
ansible all -m setup --tree /tmp/facts   //将所有主机的信息输入到/tmp/facts目录下，每台主机的信息输入到主机名文件中
```
