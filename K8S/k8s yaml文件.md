# YAML文件解析

## Pod

### 基础必填参数
```
# 1.API版本号，注意：具有多个，不同的对象可能会使用不同API
apiVersion: v1

# 2.对象类型 
kind: Pod

# 3.元数据
metadata:  
  # POD名称
  name: nginx 
  # 所属的命名空间
  namespace: default

# 4.资源内容的规范
spec:
  # 容器列表（数组）
  containers:
      # 容器名称 
    - name: nginx 
      # 容器镜像
      image: nginx 
```