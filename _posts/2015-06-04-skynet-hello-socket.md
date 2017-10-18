---
title: hello-skynet之五：hello-socket
date: 2015-06-04T10:41:41+00:00
layout: post
categories:
  - skynet
tags:
  - skynet
---
本节任务，在hello-console的基础上，

> 用snlua服务实现 socket echo服务器和客户端.

<!--more-->

关于socket的使用请参考<a href="https://github.com/cloudwu/skynet/wiki/Socket" target="_blank">skynet的wiki</a>, 这里只是用到了最简单的几个函数。

注意：从本节开始，我们将延续hello-console的思想，把skynet作为一个虚拟机，手动敲服务名（server/client）来分别加载对应的服务。

1、复制hello-console文件夹为hello-socket，删除除以下文件外的其它文件：

  * config.lua &#8211;共用配置文件
  * main.lua &#8211;共用启动入口服务

2、修改config.lua里的app_name为当前项目目录名(hello-socket)，并添加服务端监听地址参数

```lua
app_name    	= "hello-socket"
app_root    	= _root..app_name.."/"
app_server      = "0.0.0.0:7000"
```

3、服务端的服务，普通的snlua，每个客户端一个协程。协程会尽量读取客户端数据，然后打印出来。
  
```lua
local skynet    = require "skynet"
local socket    = require "socket"
--简单echo服务
function    echo(id, addr)
	socket.start(id)
	while true do
		local str = socket.read(id)
		if str then
			skynet.error("client"..id, " says: ", str)
			socket.write(id, str)
		else
			socket.close(id)
            skynet.error("client"..id, " ["..addr.."]", "disconnected")
			return
		end
	end
end
--服务入口
skynet.start(function()
    local id    = assert(socket.listen(skynet.getenv "app_server"))
    socket.start(id, function(id, addr)
        skynet.error("client"..id, " ["..addr.."]", "connected")
        skynet.fork(echo, id, addr)
    end)
end)
```

4、客户端服务，连接服务端，发送字符串，并等待服务端反馈，退出

```lua
local skynet    = require "skynet"
local socket    = require "socket"
local name = ... or ""
function _read(id)
    while true do
        local str   = socket.read(id)
        if str then
            skynet.error(id, "server says: ", str)
            socket.close(id)
            skynet.exit()
        else
            socket.close(id)
            skynet.error("disconnected")
            skynet.exit()
        end
    end
end
skynet.start(function()
    --连接到服务器
    local addr  = skynet.getenv "app_server"
    local id    = socket.open(addr)
    if not id then
        skynet.error("can't connect to "..addr)
        skynet.exit()
    end
    skynet.error("connected")
    --启动读协程
    skynet.fork(_read, id)
    socket.write(id, "hello, "..name)
end)
```

5、测试

分别打开两个终端(使用同样的命令)
  
`skynet/skynet hello-socket/config.lua`

第一个终端，输入 server 启动服务端；
  
```bash
server
[:0000000b] LAUNCH snlua server
```

第二个终端，输入 client 启动客户段；
  
```bash
client
[:0000000b] LAUNCH snlua client
[:0000000b] connected
[:0000000b] 3 server says:  hello,
[:0000000b] KILL self
```

此时服务端输出：
  
```bash
[:0000000b] client4  [127.0.0.1:58067] connected
[:0000000b] client4  says:  hello,
[:0000000b] client4  [127.0.0.1:58067] disconnected
```

可以看到客户端连接上服务端，并收到了服务端的反馈信息，然后kill self.

上面的客户没有名字，你可以试着输入 client world 给客户段起个叫&#8221;world&#8221;的名字试试看结果。

6、未涉及内容：

`<br />
socket.block             等待一个 socket 可读<br />
socket.lwrite(id, str)   低优先级队列<br />
udp                      udp<br />
dns                      非阻塞名字查询<br />
`