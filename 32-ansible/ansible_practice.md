# 32.9 ansible 最佳实践
当我们刚开始学习运用 playbook 时，可能会把 playbook 写成一个很大的文件，到后来可能你会希望这些文件是可以方便去重用的，所以需要重新去组织这些文件。ansible 支持 include 语法对 tasks, handlers, playbook 进行引用，从而我们可以对基础的通用功能进行封装，通过 "include" 对通用的功能进行组装从而实现复用。


## 1. include
### 1.1 task include

```
tasks:
  - include: wordpress.yml wp_user=timmy
  - include: wordpress.yml wp_user=alice
  - include: wordpress.yml wp_user=bob

#  Ansible 1.4 及以后的版本
tasks:
 - { include: wordpress.yml, wp_user: timmy, ssh_keys: [ 'keys/one.txt', 'keys/two.txt' ] }

 # 传递结构化变量
 tasks:
  - include: wordpress.yml
    vars:
        wp_user: timmy
        some_list_variable:
          - alpha
          - beta
          - gamma

```

### 1.2 playbook include
```
- name: this is a play at the top level of a file
  hosts: all
  remote_user: root

  tasks:

  - name: say hi
    tags: foo
    shell: echo "hi..."

- include: load_balancers.yml
- include: webservers.yml
- include: dbservers.yml
```


## 2. ansible 最佳实践
### 2.1 项目目录结构
一个完整的 ansible 项目，顶层目录结构应当包括下列文件和目录，如果你正在使用云服务，使用动态清单会更好。

```
production                # inventory file for production servers 关于生产环境服务器的清单文件
stage                     # inventory file for stage environment 关于 stage 环境的清单文件

group_vars/
   group1                 # here we assign variables to particular groups 这里我们给特定的组赋值
   group2                 # ""
host_vars/
   hostname1              # if systems need specific variables, put them here 如果系统需要特定的变量,把它们放置在这里.
   hostname2              # ""

library/                  # if any custom modules, put them here (optional) 如果有自定义的模块,放在这里(可选)
filter_plugins/           # if any custom filter plugins, put them here (optional) 如果有自定义的过滤插件,放在这里(可选)

site.yml                  # master playbook 主 playbook
webservers.yml            # playbook for webserver tier Web 服务器的 playbook
dbservers.yml             # playbook for dbserver tier 数据库服务器的 playbook

roles/
    common/               # this hierarchy represents a "role" 这里的结构代表了一个 "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```

### 2.2 playbook
通过 include 将独立分散的 ansible 任务整合在一起

```
---
# file: site.yml            # 顶层的 site
- include: webservers.yml
- include: dbservers.yml


---
# file: webservers.yml     # webservers 的配置
- hosts: webservers
  roles:
    - common
    - webtierv
```

理念是我们能够通过 “运行”(running) site.yml 来选择整个基础设施的配置.或者我们能够通过运行其子集 webservers.yml 来配置. 这与 Ansible 的 `--limit` 类似,而且相对的更为显式:

```
ansible-playbook site.yml --limit webservers
ansible-playbook webservers.yml
```

### 2.3 任务执行
```
# 想重新配置整个基础设施,如此即可:
ansible-playbook -i production site.yml

# 那只重新配置所有的 NTP 呢？太容易了.:
ansible-playbook -i production site.yml --tags ntp

# 只重新配置我的 Web 服务器呢？:
ansible-playbook -i production webservers.yml

#只重新配置我在波士顿的 Web服务器呢?:
ansible-playbook -i production webservers.yml --limit boston
```
