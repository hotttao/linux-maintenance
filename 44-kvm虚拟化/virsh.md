

virsh命令:
	虚拟机的生成需要依赖于预定义的xml格式的配置文件；其生成工具有两个：virt-manager, virt-install；

	virsh [OPTION]... COMMAND [ARG]..

	子命令的分类：
		Domain Management (help keyword 'domain')
		Domain Monitoring (help keyword 'monitor')
		Host and Hypervisor (help keyword 'host')
		Interface (help keyword 'interface')
		Networking (help keyword 'network')
		Network Filter (help keyword 'filter')
		Snapshot (help keyword 'snapshot')
		Storage Pool (help keyword 'pool')
		Storage Volume (help keyword 'volume')

	Domain Management (help keyword 'domain')
		create：从xml格式的配置文件创建并启动虚拟机；
		define：从xml格式的配置文件创建虚拟机；

		destroy：强行关机；
		shutdown：关机；
		reboot：重启；

		undefine：删除虚拟机；

		suspend/resume：暂停于内存中，或继续运行暂停状态的虚拟机；

		save/restore：保存虚拟机的当前状态至文件中，或从指定文件恢复虚拟机；

		console：连接至指定domain的控制台；

		attach-disk/detach-disk：磁盘设备的热插拔；

		attach-interface/detach-interface：网络接口设备的热插拔；
			type：bridge
			source：BRIDGE_NAME

			注意 ：无须事先创建网络接口设备；

	Domain Monitoring (help keyword 'monitor')
		domiflist
		domblklist
		...

图形管理工具：
kimchi：基于H5研发web GUI; virt-king；
OpenStack: IaaS
oVirt：
