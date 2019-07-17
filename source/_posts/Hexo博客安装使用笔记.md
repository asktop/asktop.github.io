---
title: Hexo博客安装使用笔记
date: 2019-07-17 10:58:03
tags:
  - hexo
---



官网：https://hexo.io/zh-cn/  
教程：https://blog.csdn.net/sinat_37781304/article/details/82729029

# 1. 安装 Hexo 工具
使用环境：需要安装`git`和`nodejs`。 
```
npm install hexo-cli -g
```

# 2. 创建博客项目
```
# 初始化一个 hexo 博客项目
hexo init {blog}.
# 安装依赖
cd {blog}
npm install

# 创建markdown博客文章
hexo new {docname}

# 编辑markdown博客文章
编辑 source/_posts 下生成的markdown文章
```

# 3. 博客项目目录说明
```
|——.deploy_git      # github git仓库
|——node_modules     # node模块
|——public           # hexo g 生成的文章静态页面
|——source           # 博客markdown等源文件
|    |-- _posts     # markdown博客文章目录
|——themes           # 主题包
|——_config.yml      # 配置文件
```

# 4. Hexo启动本地服务
```
# 删除本地的文章静态页面（非文章更新时，可以不执行）
hexo clean

# 将source下的文章生成静态页面
hexo generate |缩写：hexo g

# hexo启动本地服务
hexo server |缩写：hexo s

# 访问本地服务
localhost:4000
```

# 5. Hexo部署到GitHub Page服务
## 5.1 GitHub上创建公开仓库 `{username}.github.io`
**username为用户的GitHub用户名**
```
asktop.github.io
```
## 5.2 将本地项目与GitHub仓库关联
```
#修改本地配置文件 _config.yml，最后的deploy修改为
deploy:
  type: git
  repo: https://github.com/asktop/asktop.github.io.git
  branch: master
```
## 5.3 安装 deploy-git 部署工具
**用于使用 hexo deploy 命令将本地项目提交部署到GitHub**
```
npm install hexo-deployer-git --save
```
## 5.4 部署到远程GitHub Page服务
```
# 删除本地的文章静态页面（非文章更新时，可以不执行）
hexo clean

# 将source下的文章生成静态页面
hexo generate |缩写：hexo g

# 部署到GitHub远程仓库 {username}.github.io
hexo deploy |缩写：hexo d

# 访问远程服务 https://{username}.github.io
https://asktop.github.io
```

# 6. 设置个人域名
1. 在阿里云或腾讯云等购买域名 `asktop.top`，进入购买域名的服务商控制台，域名解析，添加记录：    
    a.记录类型：A  
    b.主机记录：@ 或 www 或 blog （@ 为一级域名 `asktop.top`；blog 为二级域名 `blog.asktop.top`；自定义）  
    c.解析线路：默认  
    d.记录值：192.30.252.153 或 192.30.252.154 （GitHub的服务器地址）
2. 登录GitHub——{username}.github.io项目——Settings——Options——GitHub Pages——Custom domain——设置解析的域名 `asktop.top` 或 `blog.asktop.top`。
3. 本地source文件夹下创建名为`CNAME`文件（Windows下cmd命令`type > CNAME`），编辑`CNAME`文件添加域名 `asktop.top` 或 `blog.asktop.top`。
4. 将`CNAME`文件部署到GitHub仓库
```
hexo clean
hexo g
hexo d
```
5. 访问个人域名 `asktop.top` 或 `blog.asktop.top`；访问https://asktop.github.io会自动跳转到个人域名。

# 7. 设置网站信息
**修改配置文件 _config.yml**
```
# Site
title: 问道
subtitle: 朝闻道，夕可用矣！
description:
keywords:
author: asktop
language: zh-CN
timezone:
```

# 8. 修改主题
1. 下载主题 https://hexo.io/themes
2. 将主题文件夹放在 `themes` 文件夹下
3. 修改 _config.yml 的 theme，默认为 landscape
```
# next主题：https://github.com/theme-next/hexo-theme-next
theme: next
```

# 9. 添加标签菜单

1. 新建标签页面：`$ hexo new page "tags"`

2. 编辑新建的标签页面：`source\tags\index.md`

```
title: 标签
type: tags
```

3. 修改主题 _config.yml 的 menu，默认为 landscape


```
menu:
  # 开启tags
  tags: /tags/ || tags
```

4. 文章添加参数：
```
tags:
  - 标签名1
  - 标签名2
```
