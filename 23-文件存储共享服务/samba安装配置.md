# 23.5 samba安装配置
smb: Service message block
cifs: common internet filesystem

samba：Andrew Tridgell；
  功能：
    文件系统共享；
    打印机共享；
    NetBIOS协议；

  peer/peer(workgroup model)
  domain model

程序环境：
  服务端程序包：samba，samba-common, samba-libs
    Server and Client software to interoperate with Windows machines.
  主配置文件：/etc/samba/smb.conf， 由samba-common包提供；
  主程序：
    nmbd：NetBIOS name server
    smbd：SMB/CIFS services
  Unit File：
    smb.service
    nmb.service

  监听的端口：
    137/udp, 138/udp
    139/tcp, 445/tcp

主配置文件的配置段：
```
~ ]# grep -E -i "#(====| ---)"  /etc/samba/smb.conf
#======================= Global Settings =====================================
# ----------------------- Network-Related Options -------------------------
# --------------------------- Logging Options -----------------------------
# ----------------------- Standalone Server Options ------------------------
# ----------------------- Domain Members Options ------------------------
# ----------------------- Domain Controller Options ------------------------
# ----------------------- Browser Control Options ----------------------------
# --------------------------- Printing Options -----------------------------
# --------------------------- File System Options ---------------------------
#============================ Share Definitions ==============================
```
d:\data\tools：共享，共享名（software）
  servicename:
    //172.18.0.70/software


客户端程序：
  smbclient：交互式命令行客户端，类似于lftp；
  mount.cifs：挂载cifs文件系统的专用命令；

samba的配置：
  smb.conf

    两类配置段：
      全局配置
        [global]
          Network-Related Options
            workgroup =
            server string =
            interfaces = lo eth0 192.168.12.2/24 192.168.13.2/24
            hosts allow = 127.  192.168.12.  192.168.13.
           Logging Options
            log file = /var/log/samba/log.%m
            max log size = 50
          Standalone Server Options
            security = user
              设定安全级别：取值有四个；
                share：匿名共享；
                user：使用samba服务自我管理的账号和密码进行用户认证；用户必须是系统用户，但密码非为/etc/shadow中的密码，而由samba自行管理的文件，其密码文件的格式由passdb backend进行定义；
                server：由第三方服务进行统一认证；
                domain：使用DC进行认证；基于kerberos协议进行；
            passdb backend = tdbsam
          Printing Options
            load printers = yes
            cups options = raw

      共享文件系统配置
        [SHARED_NAME]

        有三类：
          [homes]：为每个samba用户定义其是否能够通过samba服务访问自己的家目录；
          [printers]：定义打印服务；
          [shared_fs]：定义共享的文件系统；

        常用指令：
          comment：注释信息；
          path：当前共享所映射的文件系统路径；
          browseable：是否可浏览，指是否可被用户查看；
          guest ok：是否允许来宾账号访问；
          public：是否公开所有用户；
          writable：是否可写；
          read only：是否为只读；
          write list：拥有写权限的用户列表；
            用户名
            @组名
            +组名

    samba用户管理：
      smbpasswd
        smbpasswd [options] USERNAME
          -a：添加
          -x：删除
          -d：禁用
          -e：启用
      pdbedit
        -L：列出samba服务中的所有用户；
        -a, --create：添加用户为samba用户；
          -u, --user=USER：要管理的用户；
        -x, --delete：删除用户；
        -t, --password-from-stdin：从标准输出接收字符串作为用户密码；
          使用空提示符，而后将密码输入两次；

    查看服务器端的共享：
      smbclient -L SMB_SERVER  [-U USERNAME]

    交互式文件访问：
      smbclient //SMB_SERVER/SHARE_NAME [-U USERNAME]

    挂载访问：
      mount -t cifs //SMB_SERVER/SAHRE_NAME  -o username=USERNAME,password=PASSWORD

      注意：挂载操作的用户，与-o选项中指定用户直接产生映射关系；
        此时，访问挂载点，是以-o选项中的username指定的用户身份进行；本地用户对指定的路径访问，首先得拥有对应的本地文件系统权限；

  smbstatus命令：
    显示samba服务的相关共享的访问状态信息；
      -b：显示简要格式信息；
      -v：显示详细格式信息；

练习：
  创建一个共享data，路径为/var/ftp/data；要求仅centos和gentoo用户能上传；此路径对其他用户不可见；

博客实践作业：
  (1) samba server导出/data/application/web，在目录中提供wordpress;
  (2) samba  client挂载nfs server导出的文件系统至/var/www/html；
  (3) 客户端（lamp）部署wordpress，并让其正常访问；要确保能正常发文章，上传图片；
  (4) 客户端2(lamp)，挂载samba  server导出的文件系统至/var/www/html；验正其wordpress是否可被访问； 要确保能正常发文章，上传图片；

博客实践作业：
  (1) samba  server导出/data/目录；
  (2) samba  client挂载/data/至本地的/mydata目录；本地的mysqld或mariadb服务的数据目录设置为/mydata, 要求服务能正常启动，且可正常 存储数据；
    /etc/my.cnf
    [mysqld]
    datadir=/mydata

    mysql服务的数据目录的属主属组得是运行进程的用户，一般为mysql；
