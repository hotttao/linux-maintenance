# 1. 常用命令
此附录用来收集那些无法分类归总的常用命令


#### watch
`watch [-n #] COMMAND`
- 作用: 间隔执行指定的COMMAND
- `-n #`: 指定间隔时间，单位秒



#### dd
`dd if=/PATH/FROM/SRC of=/PATH/TO/DEST bs=num count=num`
- 作用: convert and copy a file
- 参数
    - `if=/path/to/src_file`: 指定源设备文件
    - `of=/path/to/dest_file`: 指定目标设备或文件
    - `bs=num`: 复制单元大小
    - `count=num`: 复制多少个 bs
- eg:
    - 破坏MBR中的bootloader： `dd if=/dev/zero of=/dev/sdb bs=256 count=1`
    - 磁盘备份: `dd if=/dev/sda of=/dev/sdb`
    - 备份MBR:  `dd if=/dev/sda  of=/tmp/mbr.back bs=512  count=1`
- 两个特殊设备:
    - `/dev/null`: 吞进所有数据，直接丢弃
    - `/dev/zero`: 泡泡机，吐零机，无限产生 0
