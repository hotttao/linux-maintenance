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


### 1.4 初始化 Master 节点
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

### 1.5 部署网络组件
初始化 Master 还有非常重要的一步，就是部署网络组件，否则各个 pod 等组件之间是无法通信的
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

kubectl get pods
```


### 1.6 k8s 集群重至
如果配置过程中出现了错误，想重新配置集群， 可以使用 `kubeadm reset` 对整个集群进行重至，然后重新使用 `kubeadm init` 进行初始化创建。但是需要注意的时，`kubeadm reset` 不会重至 flannel 网络，想要完全重至可使用以下脚本


```bash
#!/bin/bash
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
ip link delete cni0
ip link delete flannel.1
systemctl start docker
```

## 2. 安装脚本
整个集群安装比较复杂，因此我将上述过程写成了两个脚本。因此按次序执行下面脚本然后进行 `kubeadm init` 进行集群初始化即可完成配置。

### 2.1 基础环境配置脚本
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

### 2.2 镜像下载脚本
执行下面的下载脚本 `/root/kubernetes.sh`
```bash
#!/bin/bash
sudo docker login --username=1556824234@qq.com registry.cn-qingdao.aliyuncs.com
sysctl net.bridge.bridge-nf-call-ip6tables=1
sysctl net.bridge.bridge-nf-call-iptables=1

base=k8s.gcr.io
aliyun="registry.cn-qingdao.aliyuncs.com/htttao"
images=(kube-apiserver:v1.12.2 kube-controller-manager:v1.12.2 kube-scheduler:v1.12.2 kube-proxy:v1.12.2  pause:3.1  etcd:3.2.24 coredns:1.2.2)

for i in ${images[@]}
do
	docker	pull $aliyun/$i
	docker  tag  $aliyun/$i  $base/$i
done

flannel=flannel:v0.10.0-amd64
docker    pull $aliyun/$flannel
docker    tag  $aliyun/$flannel  quay.io/coreos/$flannel
```

### 2.3 集群初始化
```bash
kubeadm init --kubernetes-version=v1.12.2  --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12  --ignore-preflight-errors=Swap

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g)  $HOME/.kube/config

# Node 节点的加入集群的命令
kubeadm join 192.168.1.184:6443 --token w1b9i6.ryqstfgjmob2z8xp --discovery-token-ca-cert-hash sha256:0d3404f3919116e7efa56b2e0694c1397cd44915ed13f16d9e6e7600ada64c4c  --ignore-preflight-errors=Swap


kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

## 3. Node 节点配置
Node 节点的配置与 Master 过程类似，以此执行上述两个脚本即可，唯一的区别是在初始化时执行的是 `kubeadm join`.

```bash
# 1. 基础环境配置脚本
# 2. 执行镜像下载脚本，准备好相关镜像
# 3. 将节点加入集群, 需要注意节点的主机名不能与 Master 节点同名
kubeadm join 192.168.1.184:6443 --token w1b9i6.ryqstfgjmob2z8xp --discovery-token-ca-cert-hash sha256:0d3404f3919116e7efa56b2e0694c1397cd44915ed13f16d9e6e7600ada64c4c  --ignore-preflight-errors=Swap
```
