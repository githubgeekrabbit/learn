# golang 包管理工具go module

## go module简介

go modules 是 golang 1.11 新加的特性，用来管理模块中**包的依赖关系**。

## go mod 使用方法

- 初始化模块 `go mod init <项目模块名称>`
- 依赖关系处理 ,根据go.mod文件 `go mod tidy`
- 将依赖包复制到项目下的 vendor目录。 `go mod vendor` **如果包被屏蔽(墙),可以使用这个命令，随后使用go build -mod=vendor编译**
- 显示依赖关系 `go list -m all`
- 显示详细依赖关系 `go list -m -json all`
- 下载依赖 `go mod download [path@version]` **[path@version]是非必写的**

## 实例演示