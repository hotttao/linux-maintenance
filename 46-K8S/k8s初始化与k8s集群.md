# 46.3 k8s初始化与k8s集群


## 1. k8s 安装
### 1.1 准备 yum 源

```bash
cd /etc/yum.repo.d/
# docker-ce 源
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# kubernetes 源
vim kubernetes.repo
[kuberneters]
name=kuberneters repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=1
enabled=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg


# 设置 docker kubelet 开机自启动
systemctl enable docker kubelet
```

### 1.2 安装配置相关组件
```bash
# 安装相关组件
yum install docker-ce kubectl kubelet kubeadm

# 配置 docer 的 unit file 添加 https 代理，以便能下载相关被墙的镜像
# 不过依旧不能用，此步骤省略
# vim /usr/lib/systemd/system/docker.service # 添加
# Environment="HTTPS_PROXY=http://www.ik8s.io:10080"

systemctl daemon-reload
systemctl restart docker
docker info   # 看到 HTTPS_PROXY 行即可

# 配置 kuberneters 不受 swap 分区的影响
vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--fail-swap-on=false"

# 系统参数初始化
sysctl -w net.bridge.bridge-nf-call-ip6tables=1
sysctl -w net.bridge.bridge-nf-call-iptables=1
iptables -F
```

### 1.3 准备 kubeadm 所需镜像
因为某种不可描述的原因，kubeadm 使用到的镜像无法访问，因此需要手动准备 kubeadm 所需的镜像文件。这里有片文章可以指导你去构建相应的 镜像 https://ieevee.com/tech/2017/04/07/k8s-mirror.html

```bash
> kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.12.2
k8s.gcr.io/kube-controller-manager:v1.12.2
k8s.gcr.io/kube-scheduler:v1.12.2
k8s.gcr.io/kube-proxy:v1.12.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.2.24
k8s.gcr.io/coredns:1.2.2
```

我是自己去阿里云自建的镜像，使用下面的脚本对镜像进行重命名
```bash
#!/bin/bash
base=k8s.gcr.io
aliyun="registry.cn-qingdao.aliyuncs.com/htttao"
images=(kube-apiserver:v1.12.2 kube-controller-manager:v1.12.2 kube-scheduler:v1.12.2 kube-proxy:v1.12.2  pause:3.1  etcd:3.2.24 coredns:1.2.2)

for i in ${images[@]}
do
	docker	pull $aliyun/$i
	docker  tag  $aliyun/$i  $base/$i
done
```


### 1.3 初始化 Master 节点
```bash
kubeadm init --kubernetes-version=v1.12.2  --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12  --ignore-preflight-errors=Swap

# 运行完成之后，会提示将 Node 节点加入集群的命令
kubeadm join 192.168.1.106:6443 --token z5fqxu.dn3awhi0u5n2i6eb --discovery-token-ca-cert-hash sha256:dc333a8af6ee0c7cd1e180b43251800685b90d6338929fa508e42f76579ce50c

# 按照初始化后的提示，创建一个普通用户，并复制相应文件
# user: kubernetes
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g)  $HOME/.kube/config

# 测试
kubectl get cs
kubectl get nodes
kubectl get pods
kubectl get ns
```

### 1.4 部署网络组件
初始化 Master 还有非常重要的一步，就是部署网络组件，否则各个 pod 等组件之间是无法通信的
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get pods
```


## 2. 重启 k8s 集群
每次关机之后，k8s 集群再次开机时就会生效，因此需要重新进行配置。

### 2.1 kubeadm 重至
```bash
kubeadm reset
```

### 2.2 下载镜像
执行下面的下载脚本 `/root/kubernetes.sh`
```bash
#!/bin/bash
sudo docker login --username=1556824234@qq.com registry.cn-qingdao.aliyuncs.com
sysctl net.bridge.bridge-nf-call-ip6tables = 1
sysctl net.bridge.bridge-nf-call-iptables = 1

base=k8s.gcr.io
aliyun="registry.cn-qingdao.aliyuncs.com/htttao"
images=(kube-apiserver:v1.12.2 kube-controller-manager:v1.12.2 kube-scheduler:v1.12.2 kube-proxy:v1.12.2  pause:3.1  etcd:3.2.24 coredns:1.2.2)

for i in ${images[@]}
do
	docker	pull $aliyun/$i
	docker  tag  $aliyun/$i  $base/$i
done
```

### 2.3 重新初始化
```bash
# 系统参数初始化
sysctl -w net.bridge.bridge-nf-call-ip6tables=1
sysctl -w net.bridge.bridge-nf-call-iptables=1
iptables -F


kubeadm init --kubernetes-version=v1.12.2  --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12  --ignore-preflight-errors=Swap

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

kubeadm join 192.168.1.156:6443 --token rtkmuo.i5uvzvd9nl2b7tzi --discovery-token-ca-cert-hash sha256:b255be80f1c5861b53b3a2ec8d69fd7b1f28f9ab450fbffc6b83c1f4d688876f

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## 3. Node 节点配置
Node 节点配置与上述过程类似，只不过最后的初始化不是创建，而是加入到 master 代表的集群中。

### 3.1 基础环境配置
环境配置和软件安装跟 Master 节点没有区别，可通过下面的脚本直接完成

```bash
#!/bin/bash
# 1. 设置系统参数
mount /dev/cdrom /cdrom
iptables -F

# 2. 准备 yum 源
wget -P /etc/yum.repos.d/ https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
cat << EOF >> /etc/yum.repos.d/kubernetes.repo
[kuberneters]
name=kuberneters repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=1
enabled=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 3. 配置 kuberneters 不受 swap 分区的影响
yum install docker-ce kubelet kubeadm kubectl -y
echo 'KUBELET_EXTRA_ARGS="--fail-swap-on=false"' > /etc/sysconfig/kubelet

# 4. 启动相关服务
systemctl start docker
systemctl enable docker kubelet

cat << EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://osafqkzd.mirror.aliyuncs.com"]
}
EOF
```

### 3.2 Node 初始化
```bash
# 1. 执行镜像下载脚本，准备好相关镜像
# 2. 将节点加入集群, 需要注意节点的主机名不能与 Master 节点同名
kubeadm join 192.168.1.156:6443 --token rtkmuo.i5uvzvd9nl2b7tzi --discovery-token-ca-cert-hash sha256:b255be80f1c5861b53b3a2ec8d69fd7b1f28f9ab450fbffc6b83c1f4d688876f --ignore-preflight-errors=Swap
```
