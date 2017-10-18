---
title: simple-luago项目
date: 2015-06-12T00:21:47+00:00
layout: post
categories:
  - simple-luago
tags:
  - dev
  - github
  - go
  - lua
  - simple-luago
---
[simple-luago](https://github.com/ximenpo/simple-luago)项目专注于lua在go语言中的应用。该项目通过swig和cgo将lua虚拟机嵌入到go语言中，共有两个部分：

1. LuaAPI，适合于本来就很熟悉lua-c接口的使用者
2. LuaVM&LuaThread，提供了一个lua虚拟机的简单封装，适合于lua一般使用者

项目位于github上，地址为：<https://github.com/ximenpo/simple-luago>。
  
<!--more-->

simple-luago使用的相关应用版本为：

  * go1.4.0+ & swig3.0.5
  * go1.5.0+ & swig3.0.6+
  * swig 3.0.5+
  * lua 5.3.0+

### 支持的平台

  * macosx
  * linux(centos)
  * windows(need go1.5+)

### 安装

#### 1、准备

  * go1.4.0+ & swig3.0.5
  * go1.5.0+ & swig3.0.6+
  * git
  * gcc
  * lua5.3.0+

#### 2、安装

Macosx:
  
```bash
export GOPATH=`pwd`
go get github.com/ximenpo/simple-lua
export LUA_SRC=/path/to/lua/src
go generate github.com/ximenpo/simple-lua
go install github.com/ximenpo/simple-lua/lua
```

#### 3、测试&例子

例子都放在example目录下的，可以用以下命令运行（替换掉???的内容）。

```bash
go run src/github.com/ximenpo/simple-luago/example/lua_???.go
```
