# 32.7 ansible playbook
playbook 是 基于 yaml 语法的一种编排 ansible 命令的"脚本"，类似与 shell scritp；但是 playbook 并不是一门语言。我的理解是 playbook 就是一个配置文件，必需按照 ansible 要求的特定格式编排 ansible 的任务，这样 ansible 才能对其进行解释并执行。其能提供的功能是由 ansible 决定的。我们的目的就是学习 playbook 特定的编写要求。

相对于 ad-doc 的好处类似于 shell script 之与 shell 命令，可以重复执行，拥有更加强大的逻辑控制，因此便于执行更复杂的任务。

## 1. playbook 配置语法
下面是一个 playbook 的示例，我们将以这个示例为基础讲解如何编写 playbook。playbook 使用的 yaml 语法，因此在学习接下来的内容之前你需要先了解一下 yaml。

```
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

### 1.1 playbook的核心元素
```
- host:
  vars:
  remote_user:
  tasks:
    -
    -
    -
  variables:
    -
    -
    -
  handlers:
    -
    -

- host:

- host:
```

我们将 playbook 的配置语法分成两个部分来看，第一部分是基本的核心元素，使用这些元素我们就完全可以定义 ansible 的任务，包括
1. host: 任务要操作的主机，用法与 `ansible <host-pattern>` 选项相同
2. remote_user: 登陆的被管控主机的用户
3. tasks: 任务列表
4. handlers: 触发器，

第二部分是为了提高任务编排效率而额外提供的扩展语法包括
1. var: 变量
2. templates: 模板，模板可以利用 ansible 中的变量，为主机定义配置文件
3. when: 条件判断，比如可以依据操作系统类型决定安装什么，怎么安装模块，启动服务等
4. with_item: 循环，比如可以批量安装多个程序包，而不用定义多个任务
5. roles: 角色，抽象和独立 ansible 任务，使其可以自包含，便于移植。

### 1.2 核心元素
#### host & remote_user
host & remote_user 定义要操作的主机以及以哪个用户身份去完成要执行的步骤。host 是一个或多个组或主机的 patterns与 `ansible` 的 ` <host-pattern>` 选项使用完全一致，详细内容已经在上一节阐述在此不再累述。

```
---
- hosts: webservers
  remote_user: yourname
  sudo: yes
  sudo_user: postgres
  tasks:
  - service: name=nginx state=started
    remote_user: root
    sudo: yes
    sudo_user: root
```


#### task
task 用于定义任务列表，任务的执行是从上而下顺序执行的，且只有在所有匹配到的 host 均执行完当前的任务之后，才会继续执行下一个任务。如果某一 host 在执行任务中失败，它将会从 host 中移除，不会继续执行接下来的任务。

每个 task 的目标在于执行一个 moudle, 通常是带有特定的参数来执行.在参数中可以使用变量（variables）。

```
# 任务用于执行特定的模块，且必需具有 name
tasks:
  - name: make sure apache is running
    service: name=httpd state=running

# shell|command 执行命令的成功返回状态码非 0 时
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand || /bin/true  
    tag: run

# 使用命令
tasks:
  - name: create a virtual host file for {{ vhost }}
    template: src=somefile.j2 dest=/etc/httpd/conf.d/{{ vhost }}
```

需要注意的是还可以为每个任务定义标签，在执行 ansible-playbook 时通过 ` -t TAGS, --tags=TAGS` 选项，只运行指定标签对应的任务。


#### handlers
Handlers 也是一些 task 的列表,通过名字来引用,它们和一般的 task 并没有什么区别.Handlers 是由通知者进行 notify, 如果没有被 notify,handlers 不会执行.不管有多少个通知者进行了 notify,等到 play 中的所有 task 执行完成之后,handlers 也只会被执行一次.

Handlers 最佳的应用场景是用来重启服务,或者触发系统重启操作.除此以外很少用到了.

```
tasks
  - name: template configuration file
    template: src=template.j2 dest=/etc/foo.conf
    notify:                                # 按名称触发 handlers
       - restart memcached
       - restart apache
handlers:
  - name: restart memcached
   service:  name=memcached state=restarted
  - name: restart apache
   service: name=apache state=restarted
```

## 2. 变量与模板
为了为不同的目标主机自定义配置文件，ansible 引入了 python jinja2 的模板。通过将配置文件中与目标主机相关的配置参数(比如网卡，绑定的 ip 地址)定义成模板中变量，来达到为每个主机自定义配置文件的目的。

### 2.1 模板
ansible 中使用的模板是 python 的 jinja2，因此在创建模板之前，有必要学习一下如何定义 jinja2 模板，而将模板填充为文件，需要使用 ansible template 模块

```
task
  - name: nginx confiure
  - template: src=/var/template/nginx.j2 dest=/etc/nginx/nginx.conf
```

### 2.1 变量的定义
ansible 中变量的定义有如下几种方式:
1. Facts中生成的变量: facts 生成的是远程目标主机的所有系统信息，可通过 `ansible -m setup` 查看
2. 命令行中传递变量： `ansible-playbook  --extra-vars "name=value name=value"` 或 `--extra-vars "@some_file.json"`
3. 通过 inventory 主机清单传递变量，这种方式我们在 [32.5 ansible简介](32-ansible/ansible简介.md) 详细讲解过配置方法
4. 通过 `var` 在 playbook 中自定义变量，这种定义方式还可以将变量独立到特定的文件中
5. 通过 `role` 定义的变量，这种定义变量的方式我们会在下一节详细介绍

```
- hosts: webservers
  vars:
    - http_port: 80
  vars_files:
    - /vars/external_vars.yml
```

### 2.2 变量的作用顺序
在 ansible 中最好不要重复定义变量，保持 ansible 配置文件的简洁有助于我们维护和排错，如果相同的变量出现在不同的，其作用顺序由高到低如下所示

```
* extra vars (在命令行中使用 -e)优先级最高
* 然后是在inventory中定义的 inventory 参数(比如ansible_ssh_user)
* 接着是大多数的其它变量(命令行转换,play中的变量,included的变量,role中的变量等)
* 然后是在inventory定义的其它变量
* 然后是由系统发现的facts
* 然后是 "role默认变量", 这个是最默认的值,很容易丧失优先权
```

## 3. 逻辑控制
### 3.1 判断
#### when
ansible 中的条件判断使用 when 语句，而 when 语句的值是 Jinja2 表达式

```
tasks:
  - name: "shutdown Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_os_family == "Debian"
```

一系列的Jinja2 “过滤器” 也可以在when语句中使用, 但有些是Ansible中独有的. 比如我们想忽略某一错误,通过执行成功与否来做决定,我们可以像这样:

```
tasks:
  - command: /bin/false
    register: result
    ignore_errors: True
  - command: /bin/something
    when: result|failed
  - command: /bin/something_else
    when: result|success
  - command: /bin/still/something_else
    when: result|skipped
```

在playbooks 和 inventory中定义的变量在 when 语句中都可以使用. 下面一个例子,就是基于布尔值来决定一个任务是否被执行:

```
vars:
  - epic: true
tasks:
  - shell: echo "This certainly is epic!"
    when: epic
```

下面是 when 语句的几个常用示例

```
# 依据变量是否定义进行判断
tasks:
    - shell: echo "I've got '{{ foo }}' and am not afraid to use it!"
      when: foo is defined

    - fail: msg="Bailing out. this play requires 'bar'"
      when: bar is not defined

 # 与 with_items 一起使用
tasks:
    - command: echo {{ item }}
      with_items: [ 0, 2, 4, 6, 8, 10 ]
      when: item > 5
```

### 3，2 循环
#### with_item
ansible 中标准循环使用 with_item 语句实现，典型的使用方式如下

```
- name: add several users
  user: name={{ item.name }} state=present groups={{ item.groups }}
  with_items:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }

- name: add several users
  user: name={{ item }} state=present groups=wheel
  with_items:
     - testuser1
     - testuser2
```

除此之外， ansible 还提供了多种循环方式，迭代包括哈希表，文件列表等诸多内容。

#### 前套循环

```
- name: give users access to multiple databases
  mysql_user: name={{ item[0] }} priv={{ item[1] }}.*:ALL append_privs=yes password=foo
  with_nested:
    - [ 'alice', 'bob' ]
    - [ 'clientdb', 'employeedb', 'providerdb' ]
```

#### 对文件列表使用循环
```
---
- hosts: all

  tasks:

    # first ensure our target directory exists
    - file: dest=/etc/fooapp state=directory

    # copy each file over that matches the given pattern
    - copy: src={{ item }} dest=/etc/fooapp/ owner=root mode=600
      with_fileglob:
        - /playbooks/files/fooapp/*
```
