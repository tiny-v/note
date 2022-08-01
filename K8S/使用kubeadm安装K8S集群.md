# 使用kubeadm安装K8S集群

## 一、节点规划

| 角色 | IP | 节点名称 |
|-----|----|---------|
| master | 178.108.226.4  | master   | 
| worker | 178.108.226.9  | worker01 | 
| worker | 178.108.226.13 | worker02 | 

### 1.1 分别修改三个节点的hostname
```
hostnamectl set-hostname master
hostnamectl set-hostname worker01
hostnamectl set-hostname worker02
```

### 1.2 分别修改三个节点的 /etc/host
```
178.108.226.4 master
178.108.226.9 worker01
178.108.226.13 worker02
```

## 二、步骤

### 2.1 基础配置
```
# Centos7安装yum源 & 必备工具
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum install wget jq psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 git -y

# 关闭防火墙
systemctl disable --now firewalld

# 关闭selinux
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config

# 关闭swap
swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab 

# 安装ntpdata
rpm -ivh http://mirrors.wlnmp.com/centos/wlnmp-release-centos.noarch.rpm
yum install ntpdate -y

# 配置时间同步
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' >/etc/timezone
ntpdate time2.aliyun.com

# 加入到crontab
vi /etc/crontab
# 添加下面的定时任务
*/5 * * * *  /usr/sbin/ntpdate time2.aliyun.com

# 所有节点配置limit
ulimit -SHn 65535

vim /etc/security/limits.conf
# 末尾添加如下内容
* soft nofile 655360
* hard nofile 131072
* soft nproc 655350
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```

### 2.2 安装docker & 修改相关配置
```
# 安装docker
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum install -y docker-ce docker-ce-cli containerd.io

# 由于新版kubelet建议使用systemd，所以可以把docker的CgroupDriver改成systemd
mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

# 启动docker
systemctl enable docker && systemctl restart docker


# 使用docker info命令查看， cgroupdriver是否变成systemd
```

### 2.3 配置k8s源
```
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

### 2.4 安装kubeadm, kubelet 和 kubectl
```
yum install -y kubelet-1.22.0 kubeadm-1.22.0 kubectl-1.22.0
systemctl enable kubelet
```


### 2.5 kubeadm-config.yaml
```
# 生成k8s配置文件
kubeadm config print init-defaults --component-configs KubeProxyConfiguration,KubeletConfiguration > kubeadm-config.yaml
# 生成后可以按需修改
# 可以使用dry-run命令验证修改的语法是否正确
kubeadm init --config kubeadm-config.yaml --dry-run
# 预先拉取需要的镜像
kubeadm config images pull --config kubeadm-config.yaml
```

## 2.5 在master上操作
```
# 使用kubeadm初始化
kubeadm init \
--apiserver-advertise-address=178.108.226.4 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.0 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16

# 成功之后，执行
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



## 2.5 在worker上执行
```
# woker加入集群
kubeadm join 178.108.226.4:6443 --token 1nh1w6.rjm3h530gzyo4xwb --discovery-token-ca-cert-hash sha256:317a92fc611739445d9b30a3ec522fdcb8c33f3fa04b23c3041403295060aad6
```






