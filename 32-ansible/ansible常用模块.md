# 32.5 ansible常用模块

## 1. ansible 简介
### 1.1 程序框架
![dhcp](../images/31/ansible.jpg)


### 1.2 rpm 包组成
```
$ rpm -ql ansible |egrep -v "(python|man|doc)"
/etc/ansible
/etc/ansible/ansible.cfg       # ansible 自身的配置文件
/etc/ansible/hosts             #
/etc/ansible/roles
/usr/bin/ansible
/usr/bin/ansible-2
/usr/bin/ansible-2.7
/usr/bin/ansible-config
/usr/bin/ansible-connection
/usr/bin/ansible-console
/usr/bin/ansible-console-2
/usr/bin/ansible-console-2.7
/usr/bin/ansible-galaxy
/usr/bin/ansible-galaxy-2
/usr/bin/ansible-galaxy-2.7
/usr/bin/ansible-inventory
/usr/bin/ansible-playbook
/usr/bin/ansible-playbook-2
/usr/bin/ansible-playbook-2.7
/usr/bin/ansible-pull
/usr/bin/ansible-pull-2
/usr/bin/ansible-pull-2.7
/usr/bin/ansible-vault
/usr/bin/ansible-vault-2
/usr/bin/ansible-vault-2.7
```

## 2.ansible 常用模块

`ansible <host-pattern> [-f forks] [-m module_name] [-a args]`
  args:
    key=value

    注意：command模块要执行命令无须为key=value格式，而是直接给出要执行的命令即可；

### 2.1  常用模块：
#### command
`-m command -a 'COMMAND'`

user -a 'name= state={present|absent} system= uid='

group -a 'name= gid= state= system='

cron -a 'name= minute= hour= day= month= weekday= job= user= state='

copy -a 'dest= src= mode= owner= group='

file -a 'path= mode= owner= group= state={directory|link|present|absent} src='

ping

yum -a 'name= state={present|latest|absent}'

service -a 'name= state={started|stopped|restarted} enabled='

shell -a 'COMMAND'

script -a '/path/to/script'

setup
