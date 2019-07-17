---
title: go beego模板语法
date: 2019-07-17 11:04:16
tags:
  - golang
---

# 1. 基本语法
```go
//注释
{{/*注释*/}}
//语法体
{{代码块}}

//取当前位置的对象。在模板根级下，取responseBody对象；在遍历对象下，取子对象。
{{.}}
//取当前位置的对象属性
{{.var}}

//在子位置下取模板根级的对象
{{$.}}
//在子位置下取模板根级的对象属性
{{$.var}}

//创建变量
{{ $var := "abc"}}
//取创建的变量
{{ $var }}

//管道 pipeline
{{. | FuncA | FuncB | FuncC}}
/*
可以是上下文的变量输出，也可以是函数通过管道传递的返回值
当 pipeline 的值等于:
1、false 或 0
2、nil 的指针或 interface
3、长度为 0 的 array, slice, map, string
那么这个 pipeline 被认为是空，pipeline 为空时，相当于判断为 False
*/
```
# 2. with-end 
## 2.1 取局部作用域
```go
{{with .Request.URL}}
    //在当前对象.Request.URL下取值
    {{.Host}}
    {{.Path}}
{{end}}
```
## 2.2 对变量赋值
```go
{{with $value := "My name is %s"}}
    {{printf . "slene"}}
{{end}}
```
## 2.3 支持else
```go
{{with pipeline}}
{{else}}
    {{/* 当 pipeline 为空时会执行这里 */}}
{{end}}
```

# 3. if-else-end 分支语句
```go
{{if .IsHome}}
    //html代码
{{else if .IsAbout}}
    //html代码
{{else}}
    //html代码
{{end}}
```

# 4. range-end 循环语句
## 4.1 遍历对象，直接取
```go
//后台赋值：
pages := []struct {
    Num int
}{{10}, {20}, {30}}

this.Data["Total"] = 100
this.Data["Pages"] = pages

//前台输出：
{{range .Pages}}
    {{.Num}} of {{$.Total}}
{{end}}
/*
.Num为取.Pages子元素
$.Total为取模板中的根级上下文
*/
```

## 4.2 遍历对象，创建子对象再取
```go
//遍历数组
{{range $index, $elem := .Pages}}
    {{$index}} - {{$elem.Num}} - {{.Num}} of {{$.Total}}
{{end}}

//遍历map
{{range $key,$value := .Request.Header}}
    <p>Request.Method:{{$.Request.Method}}</p>
    <p>Keys:{{$key}}</p>
    <p>Values:{{$value}}</p>
{{end}}
```
## 4.3 支持else
```go
{{range .Pages}}
{{else}}
    {{/* 当 .Pages 为空 或者 长度为 0 时会执行这里 */}}
{{end}}
```
# 5. 基本函数
## 5.1 print 输出字符串
```go
print 对应 fmt.Sprint
printf 对应 fmt.Sprintf
println 对应 fmt.Sprintln
```
## 5.2 | 变量传递
```go
//变量可以使用符号 | 在函数间传递
{{.Name | printf "%s"}}
```
## 5.3 () 括号
```go
{{printf "nums is %s %d" (printf "%d %d" 1 2) 3}}
```
## 5.4 len 长度
```go
//返回对应类型的长度，支持类型：map, slice, array, string, chan
{{printf "The content length is %d" (.Content|len)}}
```
## 5.5 index 下标
```go
//index 支持 map, slice, array, string，读取指定类型对应下标的值
this.Data["Maps"] = map[string]string{"name": "Beego"}
{{index .Maps "name"}}
```
## 5.6 not
```go
not 返回输入参数的否定值
```
## 5.7 and
```go
//and 会逐一判断每个参数，将返回第一个为空的参数，否则就返回最后一个非空参数
{{and .X .Y .Z}}
```
## 5.8 or
```go
//or 会逐一判断每个参数，将返回第一个非空的参数，否则就返回最后一个参数
{{or .X .Y .Z}}
```
## 5.9 eq / ne / lt / le / gt / ge
```go
eq arg1 arg2: arg1 == arg2
ne arg1 arg2: arg1 != arg2
lt arg1 arg2: arg1 < arg2
le arg1 arg2: arg1 <= arg2
gt arg1 arg2: arg1 > arg2
ge arg1 arg2: arg1 >= arg2

//eq 支持多个参数，其他不支持
{{if eq true .Var1 .Var2 .Var3}}{{end}}
{{if lt 100 200}}{{end}}
```
## 5.10 call
```go
//call 可以调用函数，并传入参数。
//调用的函数需要返回 1 个值 或者 2 个值，返回两个值时，第二个值用于返回 error 类型的错误。返回的错误不等于 nil 时，执行将终止。
{{call .Field.Func .Arg1 .Arg2}}
```
# 6. beego模板函数
## 6.1 自定义模板函数
```go
//定义模板函数
func hello(in string)(out string){
    out = in + "world"
    return
}

//注册模板函数，在beego.Run()之前注册
beego.AddFuncMap("hi",hello)

//使用模板函数
{{.Content | hi}}
```
## 6.2 内置模板函数
```go
// config 获取 AppConfig 的值，可选的 configType 有 String, Bool, Int, Int64, Float, DIY
{{config configType configKey defaultValue}}

// dateformat 格式化时间，返回字符串
{{date .Time "Y-m-d H:i:s"}}
{{dateformat .Time "2006-01-02 15:04:05"}}

// compare 比较两个对象的比较，如果相同返回 true，否者 false
{{compare .A .B}}

// substr 截取字符串，支持中文截取的完美截取
{{substr .Str 0 30}}

//map_get 获取 map key的value值，可以获取二维(子)map的值
{{map_get mapName "key"}}
{{map_get mapName "key" "key"}}

// htmlquote 实现了基本的 html 字符转义
{{htmlquote .quote}}

// htmlunquote 实现了基本的反转移字符
{{htmlunquote .unquote}}

// html2str 把 html 转化为字符串，剔除一些 script、css 之类的元素，返回纯文本信息
{{html2str .Htmlinfo}}

// str2html 把相应的字符串当作 HTML 来输出，会渲染执行
{{str2html .Strhtml}}
```
# 7. 模板和layout
## 7.1 define-end 自定义模板
```go
{{define "loop"}}
    <li>{{.Name}}</li>
{{end}}
```
## 7.2 template 调用模板
```go
//导入模板，pipeline对象导入模板，在可以在模板中使用
{{template "模板名" pipeline对象}}

//调用自定义模板
<ul>
    {{range .Items}}
        {{template "loop" .}}
    {{end}}
</ul>

//调用外部模板
{{template "head.html" .}}
    //code
{{template "foot.html" .}}
```
## 7.3 自定义layout模板
**1、定义 layout_body.tpl layout框架模板**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Lin Li</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap-theme.min.css">
    <!--注册layout_head部分-->
    {{.LayoutHead}}
</head>
<body>
    <!--注册内容 body 部分-->
    <div class="container">
        {{.LayoutContent}}
    </div>
    
    <!--注册侧边按钮部分-->
    <div>
        {{.SideBar}}
    </div>
    
    <script type="text/javascript" src="http://code.jquery.com/jquery-2.0.3.min.js"></script>
    <script src="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/js/bootstrap.min.js"></script>
    
    <!--注册 js 通用脚本部分-->
    {{.Scripts}}
</body>
</html>
```
**2、定义layout框架各通用部分模板**
```html
<!--layout_head.tpl-->
<style>
     h1 {
        color: red;
     }
</style>

<!--scripts.tpl-->
<script type="text/javascript">
    $(document).ready(function() {
        // bla bla bla
    });
</script>
```
**3、注册使用layout模板**
```go
type BlogsController struct {
    beego.Controller
}

func (this *BlogsController) Get() {
    //首先解析 TplName 指定的文件
    this.TplName = "index.tpl"
    //获取文件内容赋值给Layout模板的 LayoutContent，然后最后渲染 layout_body.tpl 文件
    this.Layout = "layout/layout_body.tpl"
    this.LayoutSections = make(map[string]string)
    this.LayoutSections["LayoutHead"] = "layout/layout_head.tpl"
    this.LayoutSections["Scripts"] = "layout/scripts.tpl"
    this.LayoutSections["Sidebar"] = ""
}
```
# 8. beego集成bootstrap
```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <!-- 设置字符集 -->
    <meta charset="utf-8">
    <!-- 设置IE浏览器兼容 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <!-- 设置双核浏览器的默认浏览模式：webkit(极速模)式|ie-comp(兼容模式)|ie-stand(IE模式) -->
    <meta name="renderer" content="webkit">
    <!-- 设置视口以支持响应式布局：device-width获取设备物理宽度, scale设置缩放比例-->
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
    
    <title>文档标题标签</title>
    
    <link href="static/css/public.css" rel="stylesheet">
    <script src="static/js/public.js"></script>
</head>
<body>
    <div class="container">
		<h1>你好，世界！</h1>
	</div>
</body>
</html>
```
**static/css/public.css**
```css
/*@import url("https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css");*/
@import url("bootstrap.min.css");
```
**static/js/public.js**
```javascript
// document.write("<script src=\"https://cdn.jsdelivr.net/npm/jquery@1.12.4/dist/jquery.min.js\"></script>")
// document.write("<script src=\"https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/js/bootstrap.min.js\"></script>")
document.write("<script src=\"jquery.min.js\"></script>")
document.write("<script src=\"bootstrap.min.js\"></script>")
```


