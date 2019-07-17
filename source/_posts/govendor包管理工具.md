---
title: govendor包管理工具
date: 2019-07-17 11:03:00
tags:
  - golang
---

# 1. 安装步骤
1. 下载安装govendor工具：`go get -u github.com/kardianos/govendor`  
2. 项目必须放在GOPATH/src目录下，在项目根目录下执行`govendor init`，自动生成vendor文件夹（存放项目需要的依赖包）和vendor.json文件（有关依赖包的描述文件）。  

# 2. 常用命令
## 2.0 一般操作(在项目根目录下运行)
```
# 1、查看并下载所有未下载的包
govendor list +m
go get ...

# 2、查看并添加所有未加载到vendor中的包
govendor list +e
govendor add +e

# 3、查看并移除vendor中所有未使用的包
govendor list +u
govendor remove +u

# 4、查看并更新vendor中所有未更新的包
govendor list +v
govendor update +v
```
## 2.1 查看项目包的列表和状态  
```
#查看全部状态的所有依赖包
govendor list

#查看指定状态的所有依赖包
govendor list +m
```
```
结果示例：
状态 包名
  v  github.com/astaxie/beego
  e  gopkg.in/yaml.v2
  l  gotest/router
  l  gotest/controllers
  m  github.com/go-sql-driver/mysql
```
## 2.2 状态码
```
全称        简写
 +missing   (+m) 项目中引用却未下载到GOPATH中的包
 +external  (+e) 项目中引用且已下载到GOPATH中的包，但没加载到vendor中的包
 +unused    (+u) 已加载到vendor中但却未使用的包

 +vendor    (+v) 已加载到vendor中且正在使用的包
 +excluded  (+x) 已排除不加载到vendor中的包

 +std       (+s) Go标准库中的包
 +program   (+p) main包中的包
 +local     (+l) 项目本地的包

混合状态
 +program,local         （与）既满足+local又满足+program
 +local +vendor         （或）只要满足两者之一即可
 +vendor,program +std   vendor和program是与的关系，整体和std是或的关系
 +vendor,^program       （非）满足vendor，但却不满足program
```
## 2.3 添加GOPATH中的依赖包到vender
```
#将已下载到GOPATH中的所有依赖包添加到vendor
govendor add +e

#将已下载到GOPATH中的github.com/astaxie/beego包添加到vendor
govendor add github.com/astaxie/beego
```
## 2.4 移除vendor中的包
```
#将已加载到vendor中但未使用的所有包移除
govendor remove +u

#将已加载到vendor中指定包移除
govendor remove github.com/astaxie/beego
```
## 2.5 更新vender中的包
```
#将已加载到vendor中的所有包从GOPATH中更新到vendor
govendor update +v

#将已加载到vendor中的github.com/astaxie/beego包从GOPATH中更新到vendor
govendor update github.com/astaxie/beego
```

# 3. vendor.json示例
```
{
    "comment": "",
    "ignore": "test",
    "package": [
		{
			"checksumSHA1": "S48n6BASl14n8ogdibXvTg722IQ=",
			"path": "github.com/astaxie/beego",
			"revision": "f64e6b72e93627fa65f5f7a1b0db8ad2108a5449",
			"revisionTime": "2018-10-28T12:01:23Z"
		},
		{
			"checksumSHA1": "TRsWhl8gX4p8xoErljH/oyNmY5M=",
			"path": "github.com/garyburd/redigo/redis",
			"revision": "569eae59ada904ea8db7a225c2b47165af08735a",
			"revisionTime": "2018-04-04T16:07:26Z"
		},
		{
			"checksumSHA1": "5uj0lksF7FcX0Pgf+kjY+T0N8Mw=",
			"path": "github.com/go-sql-driver/mysql",
			"revision": "369b5d6e5e8e108ed4ae2f2b1607d444b3807dfb",
			"revisionTime": "2018-11-13T02:38:49Z"
		},
		{
			"checksumSHA1": "3JBPxvZn52LORHX9F382xpkt1sA=",
			"path": "github.com/satori/go.uuid",
			"revision": "b2ce2384e17bbe0c6d34077efa39dbab3e09123b",
			"revisionTime": "2018-10-28T12:50:25Z"
		}
    ],
    "rootPath": "gotest"
}
```