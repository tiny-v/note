## DockerCompose配置文件详解

> 这篇文章很详细： https://blog.csdn.net/linjie_830914/article/details/121490916

### 一、Docker Compose和Docker的版本对应关系
| Docker Compose |  Docker  |
| -------------- | -------- |
| 3.8 | 19.03.0+ |
| 3.7 | 18.06.0+ |
| 3.6 | 18.02.0+ |
| 3.5 | 17.12.0+ |
| 3.4 | 17.09.0+ |
| 3.3 | 17.06.0+ |
| 3.2 | 17.04.0+ |
| 3.1 | 1.13.1+ |
| 3.0 | 1.13.0+ |


### 二、Redis.yaml Demo
```
# 指定 compose 文件的版本
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
      - redisPWD=123456
    
    #
    privileged: true 
    
    # 详见下面解释
    deploy:
      resources:
         limits:
            cpus: "1.00"
            memory: 512M
         reservations:
            memory: 200M
            
    
    # 详见下面解释
    ports:
      - 6379:6379
    
    # 详见下面解释
    volumes:
      - "/opt/redis/conf/redis.conf:/etc/redis.conf"
      - "/opt/redis/logs:/var/log/redis"
      - "/opt/redis/data:/data"
    
    ulimits:
      nproc: 65535
      nofile:
        soft: 20000
        hard: 40000
    
    # 详见下面解释
    restart: always
    
    # 容器启动后, 默认执行的命令, 支持 shell 格式和 [] 格式
    command:
      redis-server /etc/redis.conf
```

### 三、常用命令解释
#### 1. restart
> 定义容器重启策略(在使用 swarm 部署时将忽略该选项, 在 swarm 使用 restart_policy 代替 restart)
```
restart:
    no                    # 禁止自动重启容器(默认)
    always                # 无论如何容器都会重启
    on-failure            # 当出现 on-failure 报错时, 容器重新启动
```


#### 2. ports
* 建立宿主机和容器之间的端口映射关系, ports 支持两种语法格式
```
 # SHORT 语法格式示例:
 
 ports:
   - "3000"                            # 暴露容器的 3000 端口, 宿主机的端口由 docker 随机映射一个没有被占用的端口
   - "3000-3005"                       # 暴露容器的 3000 到 3005 端口, 宿主机的端口由 docker 随机映射没有被占用的端口
   - "8000:8000"                       # 容器的 8000 端口和宿主机的 8000 端口建立映射关系
   - "9090-9091:8080-8081"
   - "127.0.0.1:8001:8001"             # 指定映射宿主机的指定地址的
   - "127.0.0.1:5000-5010:5000-5010"   
   - "6060:6060/udp"                   # 指定协议
```

```
# LONG 语法格式示例:(v3.2 新增的语法格式)

ports:
   # 容器端口
   - target: 80
     # 宿主机端口                    
     published: 8080 
     # 协议类型              
     protocol: tcp
     # host 在每个节点上发布主机端口,  ingress 对于群模式端口进行负载均衡                  
     mode: host                    
```

#### 3. volumes
* 定义容器和宿主机的卷映射关系, 其和 networks 一样可以位于 services 键的二级键和 compose 顶级键, 如果需要跨服务间使用则在顶级键定义, 在 services 中引用


```
# SHORT 语法格式示例:

volumes:
- /var/lib/mysql                # 映射容器内的 /var/lib/mysql 到宿主机的一个随机目录中
- /opt/data:/var/lib/mysql      # 映射容器内的 /var/lib/mysql 到宿主机的 /opt/data
- ./cache:/tmp/cache            # 映射容器内的 /var/lib/mysql 到宿主机 compose 文件所在的位置
- ~/configs:/etc/configs/:ro    # 映射容器宿主机的目录到容器中去, 权限只读
- datavolume:/var/lib/mysql     # datavolume 为 volumes 顶级键定义的目录, 在此处直接调用
```
```
# LONG 语法格式示例:(v3.2 新增的语法格式)

version: "3.2"
services:
    web:
        image: nginx:alpine
        ports:
            - "80:80"
        volumes:
            # mount 的类型, 必须是 bind、volume 或 tmpfs
            # volume 模式只指定容器路径即可, 宿主机路径随机生成;
            - type: volume 
                # 宿主机目录                 
                source: mydata
                # 容器目录              
                target: /data
                # 配置额外的选项, 其 key 必须和 type 的值相同               
                volume:                     
                    # volume 额外的选项, 在创建卷时禁用从容器复制数据
                    nocopy: true                
                        
            # bind 需要指定容器和数据机的映射路径
            - type: bind                       
                source: ./static
                target: /opt/app/static
                # 设置文件系统为只读文件系统
                read_only: true 
                            
volumes:
    # 定义在 volume, 可在所有服务中调用
    mydata:
```

#### 4.deploy-resource
* 单位: b, k, m, g 或者 kb, mb, gb
```
deploy:
  resources:
    limits:
      cpus: "1.00"
      memory: 512M
    reservations:
      memory: 200M
```

#### 5.depends_on
> 定义容器启动顺序 (此选项解决了容器之间的依赖关系， 此选项在 v3 版本中 使用 swarm 部署时将忽略该选项) 
示例：
docker-compose up 以依赖顺序启动服务，下面例子中 redis 和 db 服务在 web 启动前启动
默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系
```
version: '3'
services:
    web:
        build: .
        depends_on:
            - db      
            - redis  
    redis:
        image: redis
    db:
        image: postgres 
```









常用参数：
```

version           
    services          
        build                 # 指定包含构建上下文的路径, 或作为一个对象，该对象具有 context 和指定的 dockerfile 文件以及 args 参数值
            context               # context: 指定 Dockerfile 文件所在的路径
            dockerfile            # dockerfile: 指定 context 指定的目录下面的 Dockerfile 的名称(默认为 Dockerfile)
            args                  # args: Dockerfile 在 build 过程中需要的参数 (等同于 docker container build --build-arg 的作用)
            cache_from            # v3.2中新增的参数, 指定缓存的镜像列表 (等同于 docker container build --cache_from 的作用)
            labels                # v3.3中新增的参数, 设置镜像的元数据 (等同于 docker container build --labels 的作用)
            shm_size              # v3.5中新增的参数, 设置容器 /dev/shm 分区的大小 (等同于 docker container build --shm-size 的作用)
        
        configs               # 不知道怎么用
        cgroup_parent         # 不知道怎么用
        container_name        # 指定容器的名称 (等同于 docker run --name 的作用)
        credential_spec       # 不知道怎么用    
        devices               # 指定设备映射列表 (等同于 docker run --device 的作用)
        dns                   # 设置 DNS 地址(等同于 docker run --dns 的作用)
        dns_search            # 设置 DNS 搜索域(等同于 docker run --dns-search 的作用)
        tmpfs                 # v2 版本以上, 挂载目录到容器中, 作为容器的临时文件系统(等同于 docker run --tmpfs 的作用, 在使用 swarm 部署时将忽略该选项)
        entrypoint            # 覆盖容器的默认 entrypoint 指令 (等同于 docker run --entrypoint 的作用)
        env_file              # 从指定文件中读取变量设置为容器中的环境变量, 可以是单个值或者一个文件列表, 如果多个文件中的变量重名则后面的变量覆盖前面的变量, environment 的值覆盖 env_file 的值
            文件格式：
                RACK_ENV=development 

        environment           # 设置环境变量， environment 的值可以覆盖 env_file 的值 (等同于 docker run --env 的作用)

        expose                # 暴露端口, 但是不能和宿主机建立映射关系, 类似于 Dockerfile 的 EXPOSE 指令

        external_links        # 连接不在 docker-compose.yml 中定义的容器或者不在 compose 管理的容器(docker run 启动的容器, 在 v3 版本中使用 swarm 部署时将忽略该选项)

        extra_hosts           # 添加 host 记录到容器中的 /etc/hosts 中 (等同于 docker run --add-host 的作用)

        healthcheck           # v2.1 以上版本, 定义容器健康状态检查, 类似于 Dockerfile 的 HEALTHCHECK 指令
            test                  # 检查容器检查状态的命令, 该选项必须是一个字符串或者列表, 第一项必须是 NONE, CMD 或 CMD-SHELL, 如果其是一个字符串则相当于 CMD-SHELL 加该字符串
                NONE                  # 禁用容器的健康状态检测
                CMD                   # test: ["CMD", "curl", "-f", "http://localhost"]
                CMD-SHELL             # test: ["CMD-SHELL", "curl -f http://localhost || exit 1"] 或者　test: curl -f https://localhost || exit 1
            interval: 1m30s       # 每次检查之间的间隔时间
            timeout: 10s          # 运行命令的超时时间
            retries: 3            # 重试次数
            start_period: 40s     # v3.4 以上新增的选项, 定义容器启动时间间隔
            disable: true         # true 或 false, 表示是否禁用健康状态检测和　test: NONE 相同
        image                 # 指定 docker 镜像, 可以是远程仓库镜像、本地镜像
        init                  # v3.7 中新增的参数, true 或 false 表示是否在容器中运行一个 init, 它接收信号并传递给进程
        isolation             # 隔离容器技术, 在 Linux 中仅支持 default 值
        labels                # 使用 Docker 标签将元数据添加到容器, 与 Dockerfile 中的 LABELS 类似
        links                 # 链接到其它服务中的容器, 该选项是 docker 历史遗留的选项, 目前已被用户自定义网络名称空间取代, 最终有可能被废弃 (在使用 swarm 部署时将忽略该选项)
        logging               # 设置容器日志服务
            driver                # 指定日志记录驱动程序, 默认 json-file (等同于 docker run --log-driver 的作用)
            options               # 指定日志的相关参数 (等同于 docker run --log-opt 的作用)
                max-size              # 设置单个日志文件的大小, 当到达这个值后会进行日志滚动操作
                max-file              # 日志文件保留的数量
        network_mode          # 指定网络模式 (等同于 docker run --net 的作用, 在使用 swarm 部署时将忽略该选项)         
        networks              # 将容器加入指定网络 (等同于 docker network connect 的作用), networks 可以位于 compose 文件顶级键和 services 键的二级键
            aliases               # 同一网络上的容器可以使用服务名称或别名连接到其中一个服务的容器
            ipv4_address      # IP V4 格式
            ipv6_address      # IP V6 格式
            

        pid: 'host'           # 共享宿主机的 进程空间(PID)

        

        secrets               # 不知道怎么用

        security_opt          # 为每个容器覆盖默认的标签 (在使用 swarm 部署时将忽略该选项)

        stop_grace_period     # 指定在发送了 SIGTERM 信号之后, 容器等待多少秒之后退出(默认 10s)

        stop_signal           # 指定停止容器发送的信号 (默认为 SIGTERM 相当于 kill PID; SIGKILL 相当于 kill -9 PID; 在使用 swarm 部署时将忽略该选项)

        sysctls               # 设置容器中的内核参数 (在使用 swarm 部署时将忽略该选项)

        ulimits               # 设置容器的 limit

        userns_mode           # 如果Docker守护程序配置了用户名称空间, 则禁用此服务的用户名称空间 (在使用 swarm 部署时将忽略该选项)

        


        其他选项：
            domainname, hostname, ipc, mac_address, privileged, read_only, shm_size, stdin_open, tty, user, working_dir
            上面这些选项都只接受单个值和 docker run 的对应参数类似

        对于值为时间的可接受的值：
            2.5s
            10s
            1m30s
            2h32m
            5h34m56s
            时间单位: us, ms, s, m， h
        对于值为大小的可接受的值：
            2b
            1024kb
            2048k
            300m
            1gb
            单位: b, k, m, g 或者 kb, mb, gb
    networks          # 定义 networks 信息
        driver                # 指定网络模式, 大多数情况下, 它 bridge 于单个主机和 overlay Swarm 上
            bridge                # Docker 默认使用 bridge 连接单个主机上的网络
            overlay               # overlay 驱动程序创建一个跨多个节点命名的网络
            host                  # 共享主机网络名称空间(等同于 docker run --net=host)
            none                  # 等同于 docker run --net=none
        driver_opts           # v3.2以上版本, 传递给驱动程序的参数, 这些参数取决于驱动程序
        attachable            # driver 为 overlay 时使用, 如果设置为 true 则除了服务之外，独立容器也可以附加到该网络; 如果独立容器连接到该网络，则它可以与其他 Docker 守护进程连接到的该网络的服务和独立容器进行通信
        ipam                  # 自定义 IPAM 配置. 这是一个具有多个属性的对象, 每个属性都是可选的
            driver                # IPAM 驱动程序, bridge 或者 default
            config                # 配置项
                subnet                # CIDR格式的子网，表示该网络的网段
        external              # 外部网络, 如果设置为 true 则 docker-compose up 不会尝试创建它, 如果它不存在则引发错误
        name                  # v3.5 以上版本, 为此网络设置名称
```
