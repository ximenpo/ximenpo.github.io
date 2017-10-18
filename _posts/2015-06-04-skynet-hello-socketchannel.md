---
title: hello-skynet之六：hello-socketchannel
date: 2015-06-04T11:23:24+00:00
layout: post
categories:
  - skynet
tags:
  - skynet
---
本节在hello-socket基础上采用socketchannel，服务端不变, 客户端从socket改为socketchannel方式.

这里用的 <a href="https://github.com/cloudwu/skynet/wiki/SocketChannel" target="_blank">socketchannel</a> 只是最基本的用法，其优点和好处请查看wiki文档 <a href="https://github.com/cloudwu/skynet/wiki/SocketChannel" target="_blank">socketchannel</a>.
  
<!--more-->

1、修改配置

```bash
app_name    	= "hello-socketchannel"
app_root    	= _root..app_name.."/"
app_server_ip   = "0.0.0.0"
app_server_port = 7000
```

2、服务端对应配置修改

```lua
local id    = assert(socket.listen(
	skynet.getenv "app_server_ip",
	skynet.getenv "app_server_port"
))
```

3、客户端修改为socketchannel方式

```lua
local skynet    = require "skynet"
local sc        = require "socketchannel"
local name = ... or ""
function _do()
    --创建一个channel
    local c = sc.channel{
        host    = skynet.getenv "app_server_ip",
        port    = skynet.getenv "app_server_port",
    }
    --执行操作
    local msg = c:request("hello, "..name, function(sock)
        local str = sock:read()
        if str then
            return true, str
        else
            return false
        end
    end)
    if msg then
        skynet.error("server says: ", msg)
    else
        skynet.error("error")
    end
    c:close()
    skynet.exit()
end
skynet.start(function()
    --连接到服务器
    skynet.fork(_do)
end)
```

4、测试

参考 hello-socket 一节，内容相同。