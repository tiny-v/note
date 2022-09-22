
#### 1. 使用 <font color=Red>-o yaml </font>生成一个yaml文件
#### 2. 使用<font color=red> --dry-run </font> 不实际执行命令


* Pod
```
# 创建一个POD
kubectl run nginx --image=nginx

# 生成POD YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

* Deployment
```
# 创建一个deployment
kubectl create deployment --image=nginx nginx

# 生成deployment yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

# 将生成的yaml保存的文件里
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

# 在1.19+版本下，支持指定replicas
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```
