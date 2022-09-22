# K8S 常用命令 Kubelet

## <font color=red>get</font>(查看资源对象)

* **kubectl get** <font color=red>[资源类型]</font>： pod | deployment | node | replicaset | service | ...
```
* 查看Pod列表 - kubectl get pod

* 查看Deployment列表 - kubectl get deployment

* 查看Node列表 - kubectl get node

* 查看Replicaset列表 - kubectl get replicaset
```

* **kubectl get [资源类型]** <font color=red>[-n]</font> : 查询指定命名空间
```
* 查询所有命名空间下的资源: kubectl get pods -a

* 查询指定命名空间下的资源: kubectl get pods -n [命名空间名称]
```

* **kubectl get [资源类型]** <font color=red>[-o]</font>：格式化输出 

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

## <font color=red>describe</font>


# edit

kubectl edit replicaset [rs_name]