# 5.4 Linux特殊权限及facl扩展

## 1. 文件特殊权限
- SUID
- SGID
- Sticky
- 权限数字: SUID  SGID  Sticky -- 000-111
- eg:
    - chmod 1777 /tmp/a.txt

### 安全上下文
- 进程发起者对程序文件具有可执行权限
- 进程的属主为程序发起者
- 进程的权限取决于进程的属主


### SUID
概念:
- 进程发起者对程序文件具有可执行权限
- 进程的属主为程序文件的属主，而非程序发起者
- rws------:小写 s 表示属主原有 x 权限
- rwS------:大写 S 表示属主原有 x 权限

命令：
- chmod u+s FILE.....
- chmod u-s FILE.....

### SGID
概念:
- 默认情况下，用户创建文件时，其属组为此用户的基本组
- 一旦目录具有 SGID 权限，则对此目录具有写权限的用户，在此目录中创建的文件所属的组为目录的属组
- ---rws---: 小写 s 表示属组有 x 权限
- ---rwS---: 大写 S 表示属组没有 x 权限

命令:
- chmod g+s DIR
- chmod g-s DIR

### Sticky
概念:
- 对于一个多人可写目录，如果此目录设置了 Sticky 权限，则每个用户仅能删除自己的文件
- ------rwt: other 拥有 x 权限
- ------rwT: other 没有 x 权限

命令:
- chmod o+t DIR....
- chmod o-t DIR....
