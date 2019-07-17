---
title: go mod模块化包管理工具
date: 2019-07-17 11:01:27
tags:
  - golang
---



# 使用 go mod 原因和优势：
1. 由于 `GOPATH/src` 同一个包只能有一个版本，不同项目若需引用不同版本时会出问题；用 go mod 时，会存储项目引用包的版本信息，自动去下载或去 `GOPATH/pkg/mod` 找该版本的包。 
2. go mod 是 GO 1.11 版本后支持的，使用简单，不需要任何扩展。
3. 使用 go mod 可以在任意位置建项目，不用局限在 `GOPATH/src` 下。

# 1. mod项目创建 或 项目mod化 步骤
在任意路径下创建项目文件夹下 或 项目根目录下，执行cmd命令 `go mod init {项目名}` 进行mod初始化，然后会在项目根目录下自动生成 `go.mod` 包管理文件。  
例如：在下 `D:\gomod\testmod` 执行cmd命令 `go mod init testmod` ，显示 `go: creating new go.mod: module modtest
` 即表示成功了。

# 2. go mod 管理方式说明

| 文件名 | 功能                                        |
| ------ | ------------------------------------------- |
| go.mod | 管理引用的包 和 版本（日期和Git版本等信息） |

1. 只要项目根目录下有 `go.mod` 文件，则该项目引用包管理即是通过 go mod 方式管理的。
2. 当项目编译时，若 `go.mod` 有定义引用包的版本信息，则会去引用或下载并引用该版本的包；否则会下载最新的包放在 `GOPATH/pkg/mod` 下并将包的引用信息添加到 `go.mod` 中。
3. 在 `go.mod` 同级目录下执行的 `go get ...` 命令，会下载最新的包会放在 `GOPATH/pkg/cache` 和 `GOPATH/pkg/mod` 下，而不是 `GOPATH/src` 下。
4. go mod 模式切换：  
    go 1.11中支持通过设置临时环境变量 `set GO111MODULE=off` 来关闭 go mod 模式，则此时项目必须移动到 `GOPATH/src` 目录下，同时引用的包也不会去  `GOPATH/pkg/mod` 里找，而是 `GOPATH/src` 下找。

# 3. go mod 包引用并调用步骤  
说明：项目中引用的包，在项目编译时会自动将 go.mod 中写入版本的包和 未写入的包的最新版本下载（未写入的也会将最新版本信息写入到 go.mod 中）。  
但是，==由于开发时，未下载或未写入到 go.mod 中，无法直接调用==，所以，执行以下操作即可：

## 方式1：自动下载包或查找已下载的包并写入到 `go.mod` 中
1. 项目里需要先 import 包（已经import的可直接执行第2步）
```
import (
	...
	"github.com/asktop/gotools/atime"
	...
)
```
2. `go mod tidy` 将项目所有引用的包，自动下载最新的包到 `GOPATH/pkg/mod` 下或查找 `GOPATH/pkg/mod`最新的包，并自动写入到 `go.mod` 中
```
go mod tidy
```


## 方式2：手动下载包并手动写入到 `go.mod` 中
1. go get 包到 `GOPATH/pkg/mod` 下
```
go get [pkg]
go get github.com/asktop/gotools/atime
```
2. 将包的项目路径 和 项目版本信息 手动写入到 `go.mod` 中
```
require (
	...
	github.com/asktop/gotools v0.0.0-20190629032756-7e52dedd6814
	...
)
```

# 4. Goland 设置mod模式和被墙代理
## 4.1 Goland mod模式：
Goland开启`File——Settings——Go——Go Modules——勾选Enable...；Proxy选择direct` 。
## 4.2 被墙代理
方式1：  
Goland设置`File——Settings——Go——Go Modules——Proxy设置：https://goproxy.io`   

方式2：  
go.mod修改替换被墙地址  
```
replace (
    cloud.google.com/go => github.com/GoogleCloudPlatform/google-cloud-go latest
    golang.org/x/crypto => github.com/golang/crypto latest
    golang.org/x/net => github.com/golang/net latest
    golang.org/x/sync => github.com/golang/sync latest
    golang.org/x/sys => github.com/golang/sys latest
    golang.org/x/text => github.com/golang/text latest
    google.golang.org/appengine => github.com/golang/appengine latest
)
```

# 5. 切换到 vendor 模式
目前 go mod 各方面支持还不够完善，如果想切换回 vendor 模式，可以通过下面两种方式
```
# 临时切换
$ go mod vendor
$ go run -mod=vendor main.go

# 或设置成全局环境变量
# export GOFLAGS=-mod=vendor
$ go run main.go
```


# 6. go mod 常用命令
1. 项目module初始化  
    执行命令go mod init在当前目录下生成一个go.mod文件，执行这条命令时，当前目录不能存在go.mod文件。如果之前生成过，要先删除。
```
go mod init {项目名}
```
2. 下载并添加缺失的包以及移除不需要的包  
    执行go mod tidy命令，它会添加缺失的模块以及移除不需要的模块。执行后会生成go.sum文件(模块下载条目)。添加参数-v，例如go mod tidy -v可以将执行的信息，即删除和添加的包打印到命令行。
```
go mod tidy
go mod tidy -v
```
3. 检查当前模块的依赖是否全部下载下来，是否下载下来被修改过  
    执行命令go mod verify来检查当前模块的依赖是否全部下载下来，是否下载下来被修改过。如果所有的模块都没有被修改过，那么执行这条命令之后，会打印all modules verified。
```
go mod verify
```
4. 将所有依赖包添加到vendor中  
    执行命令go mod vendor生成vendor文件夹，该文件夹下将会放置你go.mod文件描述的依赖包，文件夹下同时还有一个文件modules.txt，它是你整个工程的所有模块。在执行这条命令之前，如果你工程之前有vendor目录，应该先进行删除。同理go mod vendor -v会将添加到vendor中的模块打印出来。
```
go mod vendor
go mod vendor -v
```