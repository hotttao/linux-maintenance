# 23.3 nfs安装配置
nfs：
  nfs: Network File System
    nis：Network Information Service；
    ldap：lightweight directory access protocol; ldap over ssl/tls；

  NFSv1
  NFSv2, NFSv3, NFSv4;

  nfsd: 2049/tcp

  辅助类的服务：rpc, portmapper
    rpc.mountd：认证；
    rpc.lockd：加锁；
    rpc.statd：状态；

    rpc: remote procedure call



  NFS  Server：
    nfs-utils：
      The nfs-utils package provides a daemon for the kernel NFS server and related tools, which provides a much higher level of performance than the traditional Linux NFS server used by most users.

    /etc/exports或/etc/exports.d/*
      /PATH/TO/SOME_DIR 	clients1(export_options, ...)  clients2(export_options, ...)
        clients：
          single host：ipv4, ipv6, FQDN；
          network：address/netmask， 支持长短格式的掩码；
          wildcards：主机名通配，例如：`*.magedu.com`;
          netgroups：NIS域内的主机组；`@group_name`；
          anonymous：使用*通配所有主机；

        General Options:
          ro：只读
          rw：读写；
          sync：同步；
          async：异步；
          secure：客户端端口小于1024，否则就要使用insecure选项；
        User ID Mapping：
           root_squash：压缩root用户，一般指将其映射为nfsnobody；
           no_root_squash：不压缩root用户；
           all_squash：压缩所有用户；
           anonuid and anongid：将压缩的用户映射为此处指定的用户；

  NFS Client:
    mount -t nfs servername:/path/to/share /path/to/mount_point  [-rvVwfnsh ] [-o options]

      # exportfs -ar
      # exportfs -au

   showmount - show mount information for an NFS server

    showmount -e NFS_SERVER_IP: 查看指定的nfs server上导出的所有文件系统；
    showmount -a：在nfs server上查看nfs服务的所有客户端列表；

  exportfs：
    exportfs
      -r：重新导出；
      -a：所有文件系统；
      -v：详细信息；
      -u：取消导出文件系统；


  其它参考文档：
    man nfs：获取nfs文件系统专用的挂载选项；


博客实践作业：
  (1) nfs server导出/data/application/web，在目录中提供wordpress;
  (2) nfs client挂载nfs server导出的文件系统至/var/www/html；
  (3) 客户端（lamp）部署wordpress，并让其正常访问；要确保能正常发文章，上传图片；
  (4) 客户端2(lamp)，挂载nfs server导出的文件系统至/var/www/html；验正其wordpress是否可被访问； 要确保能正常发文章，上传图片；

博客实践作业：
  (1) nfs server导出/data/目录；
  (2) nfs client挂载/data/至本地的/mydata目录；本地的mysqld或mariadb服务的数据目录设置为/mydata, 要求服务能正常启动，且可正常 存储数据；
