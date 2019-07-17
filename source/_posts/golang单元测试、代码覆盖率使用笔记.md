---
title: golang单元测试、代码覆盖率使用笔记
date: 2019-07-17 11:05:14
tags:
  - golang
---

# 1. 创建单元测试用例
## 1.1 创建单元测试文件
在被测试文件同目录下，创建 {filename}_test.go ，filename为被测试文件名。
## 1.2 创建单元测试方法（GoLand快捷键:Alt + Insert）
GoLand快捷键:Alt + Insert，选择要自动生成的单元测试方法类型，自定义补全方法名和测试用例内容即可。  

**单元测试方法类型：**

| 类型            | 说明                |
| --------------- | ------------------- |
| Test            | 普通 功能测试方式   |
| Table Test      | 多用例 功能测试方式 |
| Benchmark       | 普通 压力测试方式   |
| Table Benchmark | 多用例 压力测试方式 |

## 1.3 单元测试写法示例与说明
**1、被测试文件 `demo.go`**
```
package demo

//测试方法1
func demo1(a int) int {
	a++
	return a
}

//测试方法2
func demo2(a int) int {
	a++
	return a
}
```
**2、单元测试文件 `demo_test.go`**
```
package demo

import "testing"

//单用例 单元测试示例
func TestDemo1(t *testing.T) {
	//执行被测试方法
	rs := demo1(1)
	//单元测试log日志
	t.Logf("t.Logf TestDemo1 : %v", rs)
	//单元测试error日志
	t.Errorf("t.Errorf TestDemo1 : %v", rs)
}

//多用例 单元测试示例（支持并发测试）
func TestDemo2(t *testing.T) {
	//定义用例参数
	tests := []struct {
		name string
		val int
	}{
		//定义用例多场景
		{"test1", 1},
		{"test2", 2},
		{"test3", 3},
	}
	//多用例单元测试
	for _, test := range tests {
		test := test //多用例并发测试：确保多用例并发测试时，用例不会混淆
		t.Run(test.name, func(t *testing.T) {
			t.Parallel() //多用例并发测试：若不设置则按用例顺序依次执行测试
			//执行被测试方法
			rs := demo2(test.val)
			//单元测试log日志
			t.Logf("t.Logf TestDemo2 : %v", rs)
			//单元测试error日志
			t.Errorf("t.Errorf TestDemo2 : %v", rs)
		})
	}
}
```

# 2. 执行单元测试方法
## 2.1 方式1：GoLand右键执行
1、执行指定单元测试方法  
单元测试方法——右键——Run

2、执行全部单元测试方法  
单元测试文件——右键——Run

## 2.2 方式2：`go test` 命令执行
**需切换到指定包目录下执行 或 执行时指定$GOPATH下的包名 或 指定当前相对包路径**
```
# 执行当前或指定包路径下的所有单元测试方法（只打印错误日志t.Error()）
go test [package]

# 执行当前或指定包路径下的所有单元测试方法（打印执行方法详细日志）
go test -v [package]

# 执行当前或指定包路径下的指定执行文件的所有单元测试方法（打印执行方法详细日志）
go test -v [package/]{filename}_test.go

# 执行当前或指定包路径下的指定执行文件的执行单元测试方法（打印执行方法详细日志）
go test -v [package/]{filename}_test.go -run={funcname}
```

# 3. 执行代码覆盖率检测(取决于测试用例完整度)
## 3.1 方式1：GoLand右键执行
1、执行指定单元测试方法  
单元测试方法——右键——Run with Coverage

2、执行全部单元测试方法  
单元测试文件——右键——Run with Coverage

## 3.2 方式2：`go test -cover` 命令执行
**需切换到指定包目录下执行 或 执行时指定$GOPATH下的包名 或 指定当前相对包路径**
```
# 执行当前或指定包路径下的所有单元测试方法（只打印错误日志t.Error()）
go test -cover [package]

# 执行当前或指定包路径下的所有单元测试方法（打印执行方法详细日志）
go test -v -cover [package]
```
## 3.3 方式3：`go test -cover` 生成代码覆盖率检测文件
**需切换到指定包目录下执行 或 执行时指定$GOPATH下的包名 或 指定当前相对包路径**
```
# 1、执行当前或指定包路径下的所有单元测试方法，并生成代码覆盖率检测文件（打印执行方法详细日志）
go test -v -cover -coverprofile={covername} [package]

# 2、将代码覆盖率检测文件转换成html文件
go tool cover -html={covername} -o {covername}.html
```

# 4. gotests 单元测试文件自动生成工具

[gotests 单元测试文件自动生成工具](https://www.jianshu.com/p/75cb67bd44ef)

# 5. 压力测试
## 5.1 压力测试写法示例与说明
**1、被测试文件 `demo.go`**
```
package demo

//测试方法1
func demo1(a int) int {
	a++
	return a
}

//测试方法2
func demo2(a int) int {
	a++
	return a
}
```
**2、单元测试文件 `demo_test.go`**
```
package demo

import "testing"

//单用例 压力测试示例
func BenchmarkDemo1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		//执行被测试方法
		rs := demo1(1)
		//压力测试log日志
		b.Logf("b.Logf BenchmarkDemo1 : %v", rs)
		//压力测试error日志
		b.Errorf("b.Errorf BenchmarkDemo1 : %v", rs)
	}
}

//多用例 压力测试示例（依次执行）
func BenchmarkDemo2(b *testing.B) {
	//定义用例参数
	benchmarks := []struct {
		name string
		val int
	}{
		//定义用例多场景
		{"test1", 1},
		{"test2", 2},
		{"test3", 3},
	}
	//多用例压力测试
	for _, bm := range benchmarks {
		b.Run(bm.name, func(b *testing.B) {
			for i := 0; i < b.N; i++ {
				//执行被测试方法
				rs := demo2(bm.val)
				//压力测试log日志
				b.Logf("b.Logf BenchmarkDemo2 : %v", rs)
				//压力测试error日志
				b.Errorf("b.Errorf BenchmarkDemo2 : %v", rs)
			}
		})
	}
}

//多用例 压力测试示例（并发执行）
func BenchmarkDemo3(b *testing.B) {
	//定义用例参数
	benchmarks := []struct {
		name string
		val  int
	}{
		//定义用例多场景
		{"test1", 1},
		{"test2", 2},
		{"test3", 3},
	}
	//多用例压力测试
	for _, bm := range benchmarks {
		bm := bm
		b.RunParallel(func(pb *testing.PB) {
			for pb.Next() {
				//执行被测试方法
				rs := demo2(bm.val)
				//压力测试log日志
				b.Logf("b.Logf BenchmarkDemo3 : %v", rs)
				//压力测试error日志
				b.Errorf("b.Errorf BenchmarkDemo3 : %v", rs)
			}
		})
	}
}
```
## 5.2 执行压力测试
### 5.2.1 方式1：GoLand右键执行
1、执行指定压力测试方法  
压力元测试方法——右键——Run

2、执行全部压力测试方法  
压力测试文件——右键——Run

### 5.2.2 方式2：`go test -bench=. -run='^` 命令执行
**需切换到指定包目录下执行 或 执行时指定$GOPATH下的包名 或 指定当前相对包路径**
```
# 执行当前或指定包路径下的所有压力测试方法（打印执行方法详细日志）
go test -v -bench=. -run='^$' [package]

# 执行当前或指定包路径下的指定压力测试方法（打印执行方法详细日志）
go test -v -bench={benchfuncname} -run='^$' [package]
```