## GO MOD

### GO MOD 是什么？

* go mod 是Golang 1.11 版本引入的官方包（package）依赖管理工具，用于解决之前没有地方记录依赖包具体版本的问题，方便依赖包的管理。


* 在系统环境变量中配置 GO111MODULE (off, on, auto(默认值))

  * off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
  
  * on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
  
  * auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：
    * 当前目录在GOPATH/src之外且该目录包含go.mod文件
    * 当前文件在包含go.mod文件的目录下面。
    
### GO MOD 使用方法：



### 问题解决：

* Q: 用 go get 命令下载包时， 报 $GOPATH/go.mod exists but should not 

  A: GO_PATH 和 go.mod 不能共存， 可将项目中GO_PATH移除， 或删掉 go.mod 文件