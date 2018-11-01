# 2. 系统初始化脚本

```
# 1. 更改网卡开机启动，并重起网卡
sed -i "/ONBOOT/s/=no/=yes/" ifcfg-enp0s3
systemctl restart NetworkManager

# 2. 关闭 SELinux 与 iptables
sed -i "/SELINUX=enforcing/s/enforcing/permissive/" /etc/selinux/config
setenforce 0

systemctl stop firewalld
systemctl disable firewalld
iptables -F

# 3. 更改 yum 源
find /etc/yum.repos.d/ -name "*.repo" -exec  mv {} {}.back \;
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo -P /etc/profile.d/
yum install -y epel-release

# 4. chrony 时间同步服务配置
yum install -y chrony
sed -i -e "/server 3/a server ntp1.aliyun.com" -e "/^server/d" /etc/chrony.conf
systemctl restart chronyd
```
