# 14.5 grub2 系统配置与使用
上一节我们介绍了 grub 第一版的配置和使用，接下来我们学习 grub2。内容介绍同上一节相同，如下:
1. grub2 概述
  - 认识 grub2 的菜单  
  - grub2 的启动流程
2. grub2 命令行的使用
3. grub2 的配置文件
4. 安装 grub2
5. 开机过程中常见问题解决



## 2. grub2 命令行的使用
grub 2 及grub2 命令行的使用可以参考此篇博客，非常详细 https://linux.cn/article-6892-1.html 。接下来通过一个在 grub2 命令上启动操作系统的示例来演示 grub2 命令的使用，grub2 命令行支持 tab 自动补全
```
grub> ls      # 查看当前的磁盘分区设备
(hd0), (hd0, msdos1), (hd0, msdos2)

grub> set root=(hd0, msdos1)    # 设置根设备
grub> ls /                      # 查看当前设备内的文件
grub> linux /vmlinux-VERSION-releaser ro root=/dev/mapper/centos-root # 设置内核和跟目录
grub> initrd /initramfs-VErSION-releaser         # 设置 initramdisk
grub> insmod gizo                                # 装载必要的驱动模块
grub> insmod xfs
grub> insmod part_msdos
grub> boot                                      # 启动开机流程
```

## 3. grub2 的配置文件

## 4. 安装 grub2

## 5. 开机过程中常见问题解决
### 5.1 忘记开机密码
新版的 systemd 的管理机制中，需要 root 的密码才能以救援模式登陆 Linux，所以无法通过救援模式重新设置 root 密码。我们需要借助 rd.break 核心参数

1. 进入开机画面，按下 e 来进入编辑模式，在 linux16 参数上添加 rd.break 参数
![password_lose](../images/14/password_lose.jpg)
2. 改完之后按下 [crtl]+x 开始开机
3. 开机完成后屏幕会出现如下的类似画面,此时处于 RAM Disk 的环境，正真的根应该被挂载在 /sysroot
![root_remount](../images/14/root_remount.png)

```
> mount                           # 检查一下挂载点！一定会发现 /sysroot 才是对的
/dev/mapper/centos-root on /sysroot
> mount -o remount,rw /sysroot    # 挂载成可读写
> chroot /sysroot                 # 实际切换了根目录的所在！取回你的环境了
>  echo "your_root_new_pw" | passwd --stdin root  # 修改root密码
> touch /.autorelabel             # 如果SELinux=Enforcing，必需，详细说明见下
> exit
> reboot

# touch /.autorelabel 等同操作
# 1. 在 rd.break 模式下，修改完 root 密码后，将SELinux 该为 Permissive
> vim /etc/selinux/config
SELINUX=permissive

# 2. 重新开机后，重至 /etc SELinux 安全文本
> restorecon -Rv /etc
> vim /etc/selinux/config
SELINUX=Enforcing
> setenforce 1
```
SELinux 的说明:
- 在 rd.break 的 RAM Disk 环境下，系统是没有 SELinux 的,更改密码会修改 `/etc/shadow`， 所以的 shadow 的SELinux 安全本文的特性将会被取消，SELinux 为 Enforcing 的模式下，如果你没有让系统于开机时自动的回复 SELinux 的安全本文，你的系统将产生“无法登陆”的问题。加上 `/.autorelabel` 就是要让系统在开机的时候自动的使用默认的 SELinux type 重新写入 SELinux 安全本文到每个文件去


### 5.2 因文件系统错误而无法开机
文件系统错误常见原因有两个:
1. 通常就是 /etc/fstab的设置问题，尤其是使用者在实作 Quota/LVM/RAID 时，最容易写错参数， 又没有经过 mount -a 来测试挂载，就立刻直接重新开机
2. 曾经不正常关机后，也可能导致文件系统不一致情况， 也有可能会出现相同的问题

开机启动时，在检查文件系统时会有提示信息，类似下图所示。通常只要输入 root 密码进入救援模式，然后重新挂载根目录即可。
![filesystem_error](../images/14/filesystem_error.jpg)

上图属于第二种错误状况。图中的第二行处，fsck 告知其实是 `/dev/md0` 出错。系统会自动利用 fsck.ext3 去检测 /dev/md0，等到系统发现错误，并且出现“clear [Y/N]”时，输入“ y ”即可。但是需要注意的是 partition 上面的 filesystem 有过多的数据损毁时，即使 fsck/xfs_repair 完成后，可能因为伤到系统盘，导致某些关键系统文件数据的损毁，那么依旧是无法进入 Linux 此时，最好就是将系统当中的重要数据复制出来，然后重新安装，并且检验一下，是否实体硬盘有损伤的现象才好
