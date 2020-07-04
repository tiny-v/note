## 安装docker ( Centos8 安装Docker社区版(docker-ce) )


1. 安装基础工具：

	sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   
   
2. 因为在阿里云的ecs机器上面，yum镜像使用的阿里云的，找不到docker的镜像，所以需要在安装基础工具之后使用yum-config-manager来添加对应的yum仓库。 

	sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo


3. 安装最新版的 containerd.io 

	sudo yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
	
netstat -lntp  // 查看所有被监听的端口号
	
	
4. 安装docker-ce

	sudo yum install docker-ce docker-ce-cli containerd.io

	
## Docker 常用命令

### docker服务管理

*  启动 docker : 		 systemctl start docker
*  停止 docker : 		 systemctl stop docker
*  重启docker服务:        systemctl restart docker  
*  查看服务状态 :  		 systemctl status docker

### 容器生命周期管理

* start 启动一个已经停止的容器： docker start [containerId]
* exec  进入容器： docker exec -it [containerId] /bin/bash
* exit  退出容器:  退出并关闭容器： exit / Ctrl+C  |  退出不关闭容器： Ctrl+P -> Ctrl+Q
* stop  停止容器： docker stop [containerId]
* rm    删除容器:  docker rm [containerId]
* run   使用镜像构建容器并进入到容器交互命令 : docker run -itd -p port:port --name [containerName] [image]  /bin/bash
    * -d:  后台运行
    * -p (App Port):(Host Port) (指定端口号映射关系) | -P (将应用使用到的端口号映射到主机端口号上)
    * --privileged=true : container会有很大的权限，基本可以做和主机一样的事儿了
        
### 容器操作

* exec 进入容器： docker exec -it [containerId] /bin/bash
* exit 退出容器:  docker exit [containerId]

### 容器rootfs命令

### 镜像仓库

### Docker Compose 命令

* docker-compose -f [file] up -d: 构建所需要的镜像，创建网络和卷，并启动容器。

    * -f [file]: 指定配置文件，若不指定，默认查找当前路径下的 docker-compose.yml 或 docker-compose.yaml
    * -d : 后台启动服务

* docker-compose stop ： 命令会停止Compose应用相关的所有容器，但不会删除它们。 被停止的应用可通过 docker-compose restart 来重新启动
* docker-compose rm ： 用来删除已停止的compose应用。 它会删除容器和网络，但不会删除卷和镜像
* docker-compose restart : 该命令会重启已停止的compose应用， 如果用户在停止该应用后，对其做了变更，那变更的内容不会反映在重启后的应用中，这时需要重新部署应用使变更生效
* docker-compose ps : 该命令用于列出 Compose 应用中的各个容器。 输出的内容包括当前状态、容器运行的命令和网络端口
* docker-compose down: 该命令会停止并删除运行中的Compose应用。它会删除容器和网络，但不会删除卷和镜像
* docker-compose top ：该命令列出各个服务(容器)内运行的进程


### 本地镜像管理

* pull 拉取镜像到本地： docker pull [image]
* build 通过Dockerfile来打镜像： docker build -t [image]:[tag] .

### info|version
 
*  查看docker 版本:		 docker -version
*  查看docker详细信息:   docker info



## 其它知识点：

1. docker 存储驱动的选择:

   1.1  通过 docker system info 查看详细信息， 里面的Storage Driver属性为存储驱动
   
2. 设置国内镜像源

    2.1 创建或修改： /etc/docker/daemon.json
	
	Command:  vim /etc/docker/daemon.json
		 
		 # 网易的镜像仓库
		 {
			"registry-mirrors": ["http://hub-mirror.c.163.com"]
		 }
3. 1. 查询镜像版本：  创建脚本查询  /usr/local/searchImageTag.sh [image]		 

