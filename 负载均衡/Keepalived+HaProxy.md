## Keepalived + HaProxy 搭建服务器集群


### 规划
准备两台服务器

| 节点名称 | IP    |
| -----   | ----- |
|  node1  | 178.119.101.81   |
|  node2  | 178.119.101.165   |




### HaProxy 安装
在两台服务器上，分别安装HaProxy，且配置相同

1. yum -y install haproxy

2. 