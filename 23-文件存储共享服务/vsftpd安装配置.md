# 23.2 vsftpd安装配置
vsftpd：
  vsftpd is a Very Secure FTP daemon. It was written completely from scratch.

  URL：
    SCHEME://username:password@HOST:PORT/PATH/TO/FILE

    路径映射：
      用户家目录：每个用户的URL的/映射到当前用户的家目录 ；

  vsftpd以ftp用户的身份运行进程，默认认用户即为ftp用户，匿名用户的默认路径即ftp用户的家目录/var/ftp
    ftp, anonymous

    注意：一个用户通过文件共享服务访问文件系统上的文件的生效权限为此二者的交集；


  程序环境：
    主程序：/usr/sbin/vsftpd
    主配置文件：/etc/vsftpd/vsftpd.conf
    数据根目录：/var/ftp
    Systemd Unit File： /usr/lib/systemd/system/vsftpd.service

    配置vsftpd：

      用户类别：
        匿名用户：anonymous --> ftp, /var/ftp
        系统用户： 至少禁止系统用户访问ftp服务，/etc/vsftpd/ftpusers，PAM（/etc/pam.d/vsftpd）；
        虚拟用户：非系统用户，用户账号非为可登录操作系统的用户账号（非/etc/passwd）；

        用户通过vsftpd服务访问到的默认路径，是用户自己的家目录；默认可以自己有权限访问的所有路径间切换；
        禁锢用户于其家目录中；

      配置文件：/etc/vsftpd/vsftpd.conf
        directive value
        注意：directive之前不能有多余字符；

      匿名用户：
        anonymous_enable=YES
        anon_upload_enable=YES
        anon_mkdir_write_enable=YES
        anon_other_write_enable=YES

         anon_umask=077

      系统用户：
        local_enable=YES
        write_enable=YES
        local_umask=022

        辅助配置文件/etc/vsftpd/ftpusers；
          列在此文件中的用户 均禁止使用ftp服务；

        chroot_local_user=YES
          禁锢所有本地用户 于其家目录；需要事先去除用户对家目录的写权限；

        chroot_list_enable=YES
        chroot_list_file=/etc/vsftpd/chroot_list
          禁锢列表中文件存在的用户于其家目录中；需要事先去除用户对家目录的写权限；

      传输日志：
        xferlog_enable=YES
        xferlog_file=/var/log/xferlog
        xferlog_std_format=YES

      守护进程的类型：
        standalone：独立守护进程；由服务进程自行监听套按字，并接收用户访问请求；
        transient：瞬时守护进程；由受托管方代为监听套按字，服务进程没有访问请求时不启动；当托管方收到访问请求时，才启动服务进程；
          CentOS 6：xinetd独立守护进程, /etc/xinetd.d/,
          CentOS 7：由systemd代为监听；

      控制可登录vsftpd服务的用户列表：
        userlist_enable=YES
          启用/etc/vsftpd/user_list文件来控制可登录用户；
        userlist_deny=
          YES：意味着此为黑名单；
          NO：白名单；

      上传下载速率：
        anon_max_rate=0
        local_max_rate=0

      并发连接数限制：
        max_clients=2000
        max_per_ip=50

    虚拟用户：
      用户账号存储于何处？
        文件、MySQL、Redis、...

      vsftpd：认证功能托管给pam；
        基于何种存储服务来存储用户信息，以及对存储服务的驱动要靠pam实现；

      pam_mysql：
        # yum install mariadb-devel pam-devel

        # ./configure --with-pam=/usr --with-mysql=/usr --with-pam-mods-dir=/usr/lib64/security
        # make && make install

      创建数据库、授权用户、创建账号和密码；

        提供配置文件：/etc/pam.d/vsftpd.vusers
          auth required /usr/lib64/security/pam_mysql.so user=vsftpd passwd=mageedu host=127.0.0.1 db=vsftpd table=users usercolumn=name passwdcolumn=password crypt=2
          account required /usr/lib64/security/pam_mysql.so user=vsftpd passwd=mageedu host=127.0.0.1 db=vsftpd table=users usercolumn=name passwdcolumn=password crypt=2

      配置vsftpd，添加或修改以下选项：
        pam_service_name=vsftpd.vusers
        guest_enable=YES
        guest_username=vuser

      虚拟用户的写权限，通过匿名一样的指令进行定义；
        还能实现不同的用户有不同的权限；

        user_config_dir=/etc/vsftpd/vusers_config/

博客作业：pam_mysql认证ftp虚拟用户账号，且拥有不同的权限；
