---
title: skynet API：snax.lua
date: 2015-05-13T23:32:40+00:00
layout: post
categories:
  - skynet
tags:
  - skynet
---
snax 是一个方便 skynet 服务实现的简单框架。

snax对lua服务文件的接口要求：
  
```lua
--启动这个服务会调用这个函数，并把启动参数传给它。
function init( ... )
--用于响应服务退出的事件，同样能接收一些参数
function exit(...)
--远程方法，被call调用，或者obj.req.xxxxx(...)调用，函数返回值即调用返回值
response.xxxxx(...)
--远程方法，被send调用，或者obj.post.xxxx(...)调用，无返回值
accept.xxxx(...)
```
  
<!--more-->

snax文件定义的API:
  
```lua
--把服务handle转换成snax服务对象。第二个参数需要传入服务的启动名，以用来了解这个服务有哪些远程方法可以供调用。
snax.bind(handle, type)
--允许接收cluster调用
snax.enablecluster()
--退出当前服务，它等价于 snax.kill(snax.self(), ...)
snax.exit(...)
--在整个 skynet 网络中（如果你启动了多个节点），只会有一个同名服务。
snax.globalservice(name, ...)
--向 obj 提交一个热更新 patch
snax.hotfix(obj, source, ...)
--返回对name文件的接口分析的snax对象
snax.interface(name)
--让一个 snax 服务退出
snax.kill(obj, ...)
--启动服务，可以把一个服务启动多份，返回服务handle
snax.newservice(name, ...)
--输出格式化字符串到日志
snax.printf(fmt, ...)
--查询一个全局名字的服务，返回一个服务对象。如果服务尚未启动，那么一直阻塞等待它启动完毕。
snax.queryglobal(name)
--查询当前节点的具名服务，返回一个服务对象。如果服务尚未启动，那么一直阻塞等待它启动完毕。
snax.queryservice(name)
--动服务，可以把一个服务启动多份，返回snax对象
snax.rawnewservice(name, ...)
--用来获取自己这个服务对象
--等价于 snax.bind(skynet.self(), SERVER_NAME)
snax.self()
--在一个节点上只会启动一份同名服务。如果你多次调用它，会返回相同的对象。
snax.uniqueservice(name, ...)
```