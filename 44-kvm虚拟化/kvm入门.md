

## 1. KVM 组成
KVM的组件：
	kvm.ko：模块
		API
	qemu-kvm：用户空间的工具程序；
		qemu-kvm is an open source virtualizer that provides hardware emulation for the KVM hypervisor. 
	
	libvirt：工具箱，能与主机的虚拟化技术进行交互,在 qemu-kvm 基础上，更高级的管理工具。 Libvirt is a C toolkit to interact with the virtualization capabilities of recent versions of Linux (and other OSes). The main package includes the libvirtd server exporting the virtualization support.
	
		C/S：
			Client：
				libvirt-client
				virt-manager
			Daemon：
				libvirt-daemon
	
快速使用kvm技术：
	kvm 依赖硬件虚拟化，在使用前，需要检查当前系统是否支持
	# grep -E -i "(svm|vmx)" /proc/cpuinfo
	
	# yum install libvirt libvirt-daemon-kvm qemu-kvm virt-manager
	# modprobe kvm 
	# lsmod|grep kvm
	# ls /dev/kvm 

	# systemctl start libvirtd.service

	# 创建网桥(交换机)
	# virsh iface-bridge INTERFACE BRIDGE_NAME
	# virt-manager
