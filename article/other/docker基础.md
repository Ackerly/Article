# docker基础
Docker 使用 Google 公司推出的 Go语言 实现，基于 Linux 内核的cgroup、 namesapce等技术对进程进行封装隔离，由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。  
Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等，极大的简化了容器的创建和维护。使得 Docker 比虚拟机更为方便快捷  

## 基本概念
### 镜像
镜像 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。  
镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。因此，在构建镜像的时候，我们要尽量减少重复构建镜像层。  

### 容器
容器 是用镜像创建的实例，可以被启动、开始、停止、删除。容器之间相互隔离。可以把容器看做是一个简易版的Linux环境（包括root用户权限、进程空间、网络空间等）和运行在其中的应用程序  
容器与镜像的关系类似于面向对象编程中的类和对象，镜像好比是类，容器则是对象。  

### 仓库
仓库就是存放镜像的地方，我们可以把镜像发布到仓库中，需要的时候从仓库中拉下来使用。  
Docker 官方提供了 Docker Hub 仓库，提供了大量的基础镜像，方便我们下载使用  

## 安装 Docker
### macOS
使用 homebrew 安装  
``` 
brew install docker
```
手动安装链接  
``` 
https://desktop.docker.com/mac/main/amd64/Docker.dmg
```

### Windows
使用 winget 安装  
``` 
winget install Docker.DockerDesktop
```
手动安装链接  
``` 
https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe
```
安装完成之后，可以在终端看一下是否安装成功  
``` 
docker --version

Docker version 20.10.22, build 3a2c30b
```
上面这样的输出就表示安装成功了  

## 使用镜像
用 docker pull 来获取远端镜像， docker pull --help来查看具体获取方式  

### 获取镜像
``` 
docker pull nginx
```
执行成功后控制台输出如下   
``` 
Using default tag: latest
latest: Pulling from library/nginx
e9995326b091: Pull complete
71689475aec2: Pull complete
f88a23025338: Pull complete
0df440342e26: Pull complete
eef26ceb3309: Pull complete
8e3ed6a9e43a: Pull complete
Digest: sha256:47a8d86548c232e44625d813b45fd92e81d07c639092cd1f9a49d98e1fb5f737
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```
默认下载的是 latest 版本，在下载日志中我们可以看到，镜像的分层存储概念，每一层都有自己的ID，下载也是一层一层去下载的  

### 查看镜像
下载成功后执行 docker image ls 可以看到 nginx 镜像的具体信息:
镜像名称、版本、镜像下载时间、镜像大小等等  
``` 
REPOSITORY                                 TAG       IMAGE ID       CREATED         SIZE
nginx                                      latest    76c69feac34e   39 hours ago    142MB
```

### 删除镜像
使用 docker image rm <镜像Id>/<仓库名>:<标签> 来删除镜像，我们来删除上面下载的 nginx 镜像  
上面可以看到nginx镜像的ID: 76c69feac34e  
``` 
docker image rm 76c69feac34e
```
批量删除使用，docker image rm 搭配 docker image ls 使用，我们来批量删除下所有 nginx 镜像  
``` 
docker image rm $(docker image ls -q nginx)
```

### 制作镜像
镜像的制作实际上就是定制每一层所添加的配置、文件。我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像。这个脚本就是 Dockerfile。  
下面我们通过Dockerfile定制一个 nginx 镜像。  
``` 
touch Dockerfile
```
将 Dockerfile 的内容修改为  
``` 
FROM nginx
RUN echo '<div>Hello World</div>' > /usr/share/nginx/html/index.html
```
其中FROM是继承基础镜像，RUN 是执行命令行命令。我们这里给页面中输出Hello World。  
接下来就是构建镜像，让他能够跑起来，使用 docker build [OPTIONS] 路径 命令来构建镜像  
``` 
docker build -t nginx:0.0.1 .
```

## 操作容器
### 启动容器
使用 docker run [OPTIONS] IMAGE 命令启动容器，例如我们启动上面的 nginx 镜像  
-p 表示将容器80端口映射到宿主机8080端口  
-d 表示后台运行这个容器  
``` 
docker run -p 8080:80 -d nginx:0.0.1
```
测试下是否启动成功  
``` 
curl 127.0.0.1:8080
```
返回如下表示成功启动了 nginx 容器  
``` 
<div>Hello World</div>
```
### 进入容器
首先要看下正在运行的容器ID  
``` 
docker container ls

CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                  
bd075ecabbcf   nginx:v0.0.1   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp   
```
使用 docker exec [OPTIONS] 命令来进入容器，例如进入上面的 nginx 容器  
``` 
docker exec -it bd07 bash
```
其中的-i -t，是打开 Linux 命令提示符的  

### 导出容器
看下正在运行的容器ID  
``` 
docker container ls

CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                  
bd075ecabbcf   nginx:v0.0.1   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp   
```
使用 docker exec [OPTIONS] 命令来进入容器，例如进入上面的 nginx 容器
``` 
docker exec -it bd07 bash
```
其中的-i -t，是打开 Linux 命令提示符的  

### 导出容器
使用 docker export [OPTIONS] 导出容器，例如导出上面的 nginx 容器  
``` 
docker export bd07 > nginx.tar
```
### 终止容器
使用 docker container stop [OPTIONS] 来终止容器  
使用CONTAINER ID终止上面的nginx容器  
``` 
docker container stop bd07
```


原文:  
[docker基础](https://mp.weixin.qq.com/s/3qWu5bykAlLEdFipIg-iJg)
