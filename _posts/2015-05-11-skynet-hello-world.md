---
title: hello-skynet之一：hello-world
date: 2015-05-11T15:36:46+00:00
layout: post
categories:
  - skynet
tags:
  - skynet
---
最近在研究服务端相关技术，不经意间又看到了云风的<a href="https://github.com/cloudwu/skynet/wiki" target="_blank">skynet</a>。这个项目够简洁、够强大、够实用！只可惜文档和入门实例少了些。于是乎准备结合学习过程，来个学习连载，并在github上建立了相关项目：<a href="https://github.com/ximenpo/hello-skynet" target="_blank">hello-skynet</a>。

先看看skynet是什么（不要跟其他同名项目混淆了哦）：

> &#8230; 这是一个轻量级的为在线游戏服务器打造的框架。但从社区 Community 的反馈结果看，它也不仅仅使用在游戏服务器领域 &#8230;

好吧，秉承Coder大叔的习性，先来个hello-world! 功能如下：

> 打印“hello, world!”，并退出

具体步骤如下：

1、新建git库，将skynet作为子模块放到根目录下的skynet文件

2、进入skynet目录，编译skynet，参考<a href="https://github.com/cloudwu/skynet/wiki/Build" target="_blank">Build</a>

3、新建hello-world目录，并添加 config.lua 和 hello-world.lua 两个文件，内容如下：

config.lua
```lua
----------------------------------
-- 局部变量
----------------------------------
local _root = "./"
local _skynet = _root.."skynet/"
----------------------------------
-- 自定义参数
----------------------------------
app_name = "hello-world"
app_root = _root..app_name.."/"
----------------------------------
-- skynet用到的六个参数
----------------------------------
--[[
config.thread = optint("thread",8);
config.module_path = optstring("cpath","./cservice/?.so");
config.harbor = optint("harbor", 1);
config.bootstrap = optstring("bootstrap","snlua bootstrap");
config.daemon = optstring("daemon", NULL);
config.logger = optstring("logger", NULL);
--]]
-- 工作线程数
thread = 2
-- 服务模块路径（.so)
cpath = _skynet.."cservice/?.so"
-- 港湾ID，用于分布式系统，0表示没有分布
harbor = 0
-- 后台运行用到的 pid 文件
daemon = nil
-- 日志文件
logger = nil
-- 初始启动的模块
bootstrap = "snlua hello-world"
----------------------------------
-- snlua用到的参数
----------------------------------
lua_path = _skynet.."lualib/?.lua;"..app_root.."/?.lua"
lua_cpath = _skynet.."luaclib/?.so;"..app_root.."/?.so"
lualoader = "skynet/lualib/loader.lua"
luaservice = _skynet.."service/?.lua;"..app_root.."/?.lua"
-- 采用snlua bootstrap启动hello-world模块
--[[
bootstrap = "snlua bootstrap"
start = "hello-world"
--]]
```

hello-world.lua
```lua
local skynet    = require "skynet"
local skymgr    = require "skynet.manager"
local core      = require "skynet.core"
--方式1
--[[
skynet.start(function()
    --  send message to logger
    skynet.error("hello, world!")
    --  dalay and quit
    skynet.timeout(1, skymgr.abort)
end)
--]]
--方式2
--[－[
skynet.start(function()
    -- get the logger service
    local logger    = skynet.localname(".logger")
    if not logger then
        skynet.error("no logger found")
        skymgr.abort()
    end
    core.send(logger, skynet.PTYPE_TEXT, 0, "hello, world!")
    --  delay quit for the logger display
    skynet.timeout(1, function()
        skymgr.abort()
        --skynet.exit()
    end)
end)
--]]
```

4、测试

进入git库根目录，敲入代码测试：
  
```bashstudy-skynet simple$ skynet/skynet hello-world/config.lua```

如果一切正常的话，屏幕上将会出现类似下面的文字
  
```bash
study-skynet simple$ skynet/skynet hello-world/config.lua
[:00000001] LAUNCH logger
[:00000002] LAUNCH snlua hello-world
[:00000002] hello, world!
study-skynet simple$
```

5、内容说明

<pre>
*   输出错误信息：           skynet.error(...)
*   获取本地服务句柄方式：    skynet.localname(...)
*   设置定时器方式：         skynet.timeout(...)
*   skynet强制退出方式：     skyname.abort()
*   服务开始方式：           skynet.start(...)
*   服务注销方式：           skynet.exit()
*   发送原始文本消息方式：    skynet.core.send(...)
</pre>

6、思考

这里的采用的是直接 snlua hello-world 方式，即hello-world.lua作为第一个启用的服务。你可以尝试换成缺省的 snlua bootstrap 方式，观察下输出有什么不同，为什么？

备注：取消 config.lua 最后一段的注释。
