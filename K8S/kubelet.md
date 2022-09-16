## K8S 常用命令 Kubelet

### 1. kubectl get [资源类型] **[-n]** [-o]

* [资源类型]： pods | deployments | nodes
```
* 查看Pods列表 - kubectl get pods 

* 查看Deployment列表 - kubectl get deployments

* 查看nodes列表 - kubectl get nodes

* 查看replicasets列表 - kubectl get replicasets
```

* **[-n]**: 查询指定命名空间
```
* 查询所有命名空间下的资源: kubectl get pods -a

* 查询指定命名空间下的资源: kubectl get pods -n [命名空间名称]
```

* [-o]：格式化输出 

| 命令 | 含义|
| ------------------------ | ------------------------------ |
| -o=custom-columns=自定义列名 | 根据自定义列名进行输出，以逗号分隔 | 
| -o=custom-columns-file=文件名 | 从文件中获取自定义列名进行输出 |
| -o=json                  |	以JSON格式显示结果     |
| -o=jsonpath=<template> |	输出jsonpath表达式定义的字段信息 |
| -o=jsonpath-file=文件名 |	输出jsonpath表达式定义的字段信息，来源于文件 |
| -o=name	| 仅输出资源对象的名称 |
| -o=wide	| 输出额外信息。对于Pod，将输出Pod所在的Node名 |
| -o=yaml	| 以yaml格式显示结果 |