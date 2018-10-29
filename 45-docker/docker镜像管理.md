# 45.3 Docker 镜像管理
镜像依赖于特定的文件系统，实现"分层构建，联合挂载"。特殊文件系统目前有两种实现:overlay2, aufs


## 1. 镜像获取
`docker pull <registry>[:<port>]/[namespaces/]<name>:<tag>`

## 2. 镜像制作
### 2.1 基于 Dockerfile

### 2.2 基于容器制作
启动容器后，然后在容器中作好所需要的修改，比如使用 yum 安装上特定的软件，或修改特定程序的配置文件。最后使用 `docker commit` 将容器最上面的可写层制作成一个新的镜像。

#### docker commit
`docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`

```
# 基于 busybox 制作一个简单的 httpd 镜像
docker commit -a "tao" -c 'CMD ["/bin/httpd", "-f", "-h", "/data/html"]' -p  b1 tao:v2

```

### 2.3 Docker Hup automated builds

## 3. 镜像共享
#### docker tag

#### docker push

#### docker login

#### docker save

#### docker load
