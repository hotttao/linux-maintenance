# 32.7 ansible playbook

## 1. playbook 配置语法
playbook的核心元素：
  tasks: 任务
  variables: 变量
  templates: 模板
  handlers: 处理器
  roles: 角色

变量：
  facts
  --extra-vars "name=value name=value"
  role定义
  Inventory中的变量：
    主机变量
      hostname name=value name=value
    组变量
      [groupname:vars]
      name=value
      name=value

Inventory的高级用法：

Playbook：

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

"ansible_distribution_major_version": "7",

nginx.conf
  worker_processes {{ ansible_processor_cores * ansible_processor_count - 1 }};
