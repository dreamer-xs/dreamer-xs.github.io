---
layout: post
title:  "Windows 平台下 Go 语言的安装和环境变量设置"
date:   2017-07-03 15:31:01 +0800
categories: go
tag: go部署
---

* content
{:toc}


******

# 下载Go语言安装包
>[官网地址](https://golang.org/dl/)

# 安装Go语言安装包
>默认安装到`C:\Go\`，也可自行设置

# Go语言环境变量设置
##### 在windows环境变量中，将前面安装的go命令添加到系统path中

##### 设置Go 工作目录 GOPATH
>新建系统变量 GOPATH，将其指向你的代码目录

##### 命令行对环境变量进行验证
>打开cmd命令行，输入 go env 查看环境变量进行验证是否设置成功
![]({{ '/styles/images/go_env.png' | prepend: site.baseurl  }})
