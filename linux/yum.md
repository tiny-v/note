
### yum 常用命令
```
1.使用YUM查找软件包
命令：yum search ~
2.列出所有可安装的软件包
命令：yum list
3.列出所有可更新的软件包
命令：yum list updates
4.列出所有已安装的软件包
命令：yum list installed
5.列出所有已安装但不在Yum Repository 內的软件包
命令：yum list extras
6.列出所指定软件包
命令：yum list ～
7.使用YUM获取软件包信息
命令：yum info ～
8.列出所有软件包的信息
命令：yum info
9.列出所有可更新的软件包信息
命令：yum info updates
10.列出所有已安裝的软件包信息
命令：yum info installed
11.列出所有已安裝但不在Yum Repository 內的软件包信息
命令：yum info extras
12.列出软件包提供哪些文件
命令：yum provides~
13.安装软件
命令：yum install ~
14.删除软件
命令: yum remove ~
```

### 删除yum安装的软件
```
# 进入yum源目录
cd /etc/yum.repos.d && ll

# 删除源文件(.repo)
rm -f ${文件名}.repo
```
