# 32.8 ansible role
role 角色，基于一个已知的文件结构，去自动的加载某些 vars_files，tasks 以及 handlers。基于 roles 对内容进行分组，使得我们可以容易地与其他用户分享 roles。

roles 是 playbook 的一个独立自包含目录，包含了执行 playbook 任务所有的配置文件及被操作文件，使得 playbook 的执行不需要依赖于任何外部环境。

## 1. roles 使用
### 1.1 roles 目录结构
一个包含 roles 的典型项目结构如下所示，如果你在 playbook 中同时使用 roles 和 tasks，vars_files 或者 handlers，roles 将优先执行。

```
webservers.yml
roles/
   webservers/        # 与 playbook 对应的 roles 目录
     files/          # copy tasks，script tasks，引用 roles/x/files/ 中的文件无需指明路经
     templates/      # template tasks 可以引用 roles/x/templates/ 中的文件，不需要指明文件的路径

     tasks/          # main.yml 存在, 其中列出的 tasks 将被添加到 webservers.yml 中
     handlers/       # main.yml 存在, 其中列出的 handlers 将被添加到 play 中
     vars/           # main.yml 存在, 其中列出的 variables 将被添加到 play 中
     defaults/       # main.yml 用于定义默认变量，这些变量在所有可用变量中拥有最低优先
     meta/           # main.yml 存在, 其中列出的 “角色依赖” 将被添加到 roles 列表中 (1.3 and later)

```

### 1.2 roles 使用
角色的使用，只需在 playbook 的 roles 语句中添加角色即可

```
# vim webservers.yml
---
- hosts: webservers
  roles:
     - common
     - webservers
```

也可以使用参数化的 roles，这种方式通过添加变量来实现

```
---

- hosts: webservers
  roles:
    - common
    - { role: foo_app_instance, dir: '/opt/a',  port: 5000 }
    - { role: foo_app_instance, dir: '/opt/b',  port: 5001 }
```


也可以为 roles 设置触发条件

```
---

- hosts: webservers
  roles:
    - { role: some_role, when: "ansible_os_family == 'RedHat'" }
```

最后，也可以给 roles 分配指定的 tags。比如:

```
---

- hosts: webservers
  roles:
    - { role: foo, tags: ["bar", "baz"] }
```

### 1.3 执行顺序
如果 play 同时包含 tasks 和roles，这些 tasks 将在所有 roles 应用完成之后才被执行。如果你希望定义一些 tasks，让它们在 roles 之前以及之后执行，你可以这样做:

```
---

- hosts: webservers

  pre_tasks:
    - shell: echo 'hello'

  roles:
    - { role: some_role }

  tasks:
    - shell: echo 'still busy'

  post_tasks:
    - shell: echo 'goodbye'
```

### 1.4 角色依赖
角色依赖可以自动地将其他 roles 拉取到现在使用的 role 中。角色依赖保存在 roles 目录下的 `meta/main.yml` 文件中。这个文件应包含一列 roles 和 为之指定的参数

```
---
dependencies:
  - { role: common, some_parameter: 3 }
  - { role: apache, port: 80 }
  - { role: postgres, dbname: blarg, other_parameter: 12 }
```

“角色依赖” 总是在 role （包含”角色依赖”的role）之前执行，并且是递归地执行。默认情况下，作为 “角色依赖” 被添加的 role 只能被添加一次，如果另一个 role 将一个相同的角色列为 “角色依赖” 的对象，它不会被重复执行。但这种默认的行为可被修改，通过添加 allow_duplicates: yes 到 meta/main.yml 文件中。

```
# wheel 角色中
---
allow_duplicates: yes
dependencies:
- { role: tire }
- { role: brake }

# 引用 wheel 的其他角色
---
dependencies:
- { role: wheel, n: 1 }
- { role: wheel, n: 2 }
- { role: wheel, n: 3 }
- { role: wheel, n: 4 }
```
