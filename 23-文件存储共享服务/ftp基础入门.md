# 23.1 ftp基础入门



ftp：
	ftp：file transfer protocol，文件传输协议 ；
		两类连接：
			命令连接：传输命令
			数据连接：传输数据
				两种模式：
					主动模式：PORT
						Server: 20/tcp连接客户端的命令连接使用的端口向后的第一个可用端口；
					被动模式：PASV
						Server：打开一个随机端口，并等待客户端连接

	PAM：Pluggable Authenticate Module
		认证框架：库，高度模块化；

	协议：C/S
		Server：
			Windows: Serv-U, IIS, Filezilla
			开源：wuftpd, proftpd, pureftpd, vsftpd(Very Secure FTP daemon), ...
		Client：
			Windows：ftp, Filezilla, CuteFTP, FlashFXP, ...
			开源：lftp, ftp, Filezilla, gftp, ...
