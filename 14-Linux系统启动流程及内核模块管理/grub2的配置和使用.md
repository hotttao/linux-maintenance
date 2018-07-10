# 14.5 grub2 系统配置与使用
上一节我们介绍了 grub 第一版的配置和使用，接下来我们学习 grub2。内容介绍同上一节相同，如下:
1. grub2 概述
  - 认识 grub2 的菜单  
  - grub2 的启动流程
2. grub2 命令行的使用
3. grub2 的配置文件
4. 安装 grub2
5. 开机过程中常见问题解决

## 3. grub2 命令行的使用
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
