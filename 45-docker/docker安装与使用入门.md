# 45.1 docker 安装与使用入门


## 1. docker 简介
### 1.1 docker 架构
![docker-archi](../images/45/docker_architecture.jpg)
C/S 架构的服务，restful 风格的 api 接口。

#### register
一个应用程序
1. 镜像存储的仓库
	- 一个 register 有多个仓库，一个仓库只放一个应用程序，包含一个应用的多个版本
	- 镜像标识: 仓库名:标签，eg: nginx:1.10
2. 用户认证
3. 当前可用镜像的索引


### 1.2 docker 安装
#### 依赖环境
- 64 bits CPU
- Linux Kernel 3.10+
- Linux Kernel cgroups and namespaces

#### 安装
```
# 1. 配置 yum 源
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo yum-config-manager --enable docker-ce-edge
sudo yum-config-manager --enable docker-ce-test

# 2. 安装
yum install docker-ce

# 3. 配置镜像加速器
vim /etc/docker/daemon.json
{
	"registry-mirrors": ["https://registry.docker.cn.com"]
}

# 4. 启动服务
systemctl start docker.service
```


## 2. docker 使用入门
docker 有多个对象，对每个对象支持增删改查。常用的对象包括 images,containers, networks, volumes, plugins


