## 部署redis

### 一、步骤

* 创建目录并修改权限
```
mkdir -p /opt/redis/data
mkdir -p /opt/redis/logs
mkdir -p /opt/redis/conf
chmod +777 /opt/redis/data/*
chmod +777 /opt/redis/logs/*
chmod +777 /opt/redis/conf/redis.conf
```

* 将下面 redis.yaml 放到 /opt/redis/ 下，  redis.conf 放到 /opt/redis/conf/ 下

* 下载镜像： docker pull redis:7.0

* docker-compose -f /opt/redis/redis.yaml up -d


### 二、附件

#### redis.yaml
```
version: "3.2"
# 定义所有的service信息, services下第一级别的key就是service的名称(比如redis)
services:
  # Service名称
  redis:
    # 指定 docker 镜像, 可以是远程仓库镜像、本地镜像
    image: redis:7.0
    hostname: redis
    # 指定容器的名称 (等同于 docker run --name 的作用)
    container_name: redis
 
    # 配置docker container的环境变量
    # environment 的值可以覆盖 env_file 的值 (等同于 docker run --env 的作用)
    environment:
      - TZ=Asia/Shanghai
    privileged: true   
    deploy:
      resources:
         limits:
            cpus: "1.00"
            memory: 512M
         reservations:
            memory: 200M
    ports:
      - 6379:6379
    volumes:
      - "/opt/redis/conf/redis.conf:/etc/redis.conf"
      - "/opt/redis/logs:/var/log/redis"
      - "/opt/redis/data:/data"   
    ulimits:
      nproc: 65535
      nofile:
        soft: 20000
        hard: 40000
    restart: always    
    # 容器启动后, 默认执行的命令, 支持 shell 格式和 [] 格式
    command:
      redis-server /etc/redis.conf
```

#### redis.conf 
```
### 指定redis绑定的主机地址，注释掉这部分，使redis可以外部访问
# bind 127.0.0.1 -::1

### 指定访问redis服务端的端口
port 6379

### 指定客户端连接redis服务器时，当闲置的时间为多少（如300）秒时关闭连接（0表示禁用）
timeout 0

### 默认情况下，Redis不作为守护进程运行。如果需要，请使用“yes”
daemonize no

### 给redis设置密码，不需要密码的话则注释
requirepass 123456

### 开启redis持久化，默认为no
appendonly yes

### 防止出现远程主机强迫关闭了一个现有的连接的错误 默认是300
tcp-keepalive 300

### 指定redis数据库的日志级别，常用的日志级别有debug、verbose、notice、warning，不进行修改的情况下默认的是notice
loglevel notice

### 指定redis数据库多长时间内（s）有多少次（c）更新操作时就把缓存中的数据同步到本地库，比如：save 600 2，指的是10分钟内有2次更新操作，就同步到本地库
#save <s><c>

### 指定redis的最大内存。由于Redis 在启动时会把数据加载到内存中，当数据达到最大内存时，redis会自动把已经到期和即将到期的key值。所以可以根据需求调整自己的所需的最大内存
maxmemory 300mb

### 设置了maxmemory的选项，redis内存使用达到上限。可以通过设置LRU算法来删除部分key，释放空间。默认是按照过期时间的,如果set时候没有加上过期时间就会导致数据写满maxmemory
maxmemory-policy volatile-lru

### 设置外部网络连接redis服务，开启需配置bind ip或者设置访问密码，关闭此时外部网络可以直接访问
# protected-mode yes

### 设置日志文件
logfile /var/log/redis/redis.log
```

