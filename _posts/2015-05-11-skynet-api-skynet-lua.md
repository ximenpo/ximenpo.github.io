---
title: skynet API：skynet.lua
date: 2015-05-11T18:29:46+00:00
layout: post
categories:
  - skynet
tags:
  - skynet
---
skynet消息类型常量：
  
```lua
skynet.PTYPE_TEXT = 0,
skynet.PTYPE_RESPONSE = 1,
skynet.PTYPE_MULTICAST = 2,
skynet.PTYPE_CLIENT = 3,
skynet.PTYPE_SYSTEM = 4,
skynet.PTYPE_HARBOR = 5,
skynet.PTYPE_SOCKET = 6,
skynet.PTYPE_ERROR = 7,
skynet.PTYPE_QUEUE = 8,	-- used in deprecated mqueue, use skynet.queue instead
skynet.PTYPE_DEBUG = 9,
skynet.PTYPE_LUA = 10,
skynet.PTYPE_SNAX = 11,
```

<!--more-->

skynet可用方法：
  
```lua
--  把一个地址数字转换为一个可用于阅读的字符串
skynet.address(addr)
--  判断是否有服务死锁 c.command("ENDLESS")
skynet.endless()
--  code cache，详见： https://github.com/cloudwu/skynet/wiki/CodeCache
--  可以调用 skynet.cache.clear() 清理缓存以方便热更新
skynet.cache
--  Req/Rsp模式，阻塞型API。用lua pack方式发送数据，并将接收到的数据unpack并返回
skynet.call(addr, typename, ...)
--  注册特定类消息的处理函数
skynet.dispatch(typename, func)
--  ?
skynet.dispatch_unknown_request(unknown)
--  ?
skynet.dispatch_unknown_response(unknown)
--  输出错误信息
skynet.error(...)
--  服务退出，含c.command("EXIT")
--  skynet.exit 之后的代码都不会被运行。而且，当前服务被阻塞住的 coroutine 也会立刻中断退出。这些通常是一些 RPC 尚未收到回应。
skynet.exit()
--  从功能上，它等价于 skynet.timeout(0, function() func(...) end)
--  但是比 timeout 高效一点。因为它并不需要向框架注册一个定时器。
skynet.fork(func,...)
--  生成一个唯一 session 号
skynet.genid()
--  获取配置参数
skynet.getenv(key)
--  返回addr对应的harbor id
skynet.harbor(addr)
--  在 skynet.start 注册的函数之前执行，用于在start前执行skynet阻塞型API
skynet.init(f, name)
--  根据名字获取本地服务句柄 c.command("QUERY", name)
skynet.localname(name)
--  获取消息队列长度
skynet.mqlen()
--  用于启动一个新的 Lua 服务。name 是脚本的名字（不用写 .lua 后缀）。只有被启动的脚本的 start 函数返回后，这个 API 才会返回启动的服务的地址，这是一个阻塞 API 。
--  如果被启动的脚本在初始化环节抛出异常，或在初始化完成前就调用 skynet.exit 退出，｀skynet.newservice` 都会抛出异常。
--  如果被启动的脚本的 start 函数是一个永不结束的循环，那么 newservice 也会被永远阻塞住。
skynet.newservice(name, ...)
--  返回 skynet 节点进程启动的时间 c.command("NOW")
--  每 100 表示真实时间 1 秒。这个函数的开销小于查询系统时钟。
--  在同一个时间片内（没有因为阻塞 API 挂起），这个值是不变的。
skynet.now()
--  打包lua对象, 返回一个lightuserdata和一个长度
skynet.pack
--  打包lua对象，返回一个字符串
skynet.packstring
--  xpcall start 函数
skynet.pcall(start)
--  查询服务, 如果还没有创建过目标服务则一直等下去，直到目标服务被(其他服务触发而)创建
--  [global为可选参数]
skynet.queryservice(global, ...)
--  每次调用都可以得到一个新的临界区。用于保护一段代码不被同时运行。
--  内部采用协程队列的方式，按排序执行入队的协程。
skynet.queue()
--  和 skynet.call 功能类似（也是阻塞 API）,发送raw数据，返回raw数据
skynet.rawcall(addr, typename, msg, sz)
--  和 skynet.send 功能类似，但更细节一些。
--  它可以指定发送地址（把消息源伪装成另一个服务），指定发送的消息的 session 。
--  注：address 和 source 都必须是数字地址，不可以是别名。
skynet.redirect(dest,source,typename,...)
--  注册协议，缺省支持 lua response error
--  协议名(name)
--  协议ID(id)
--  发送消息的打包函数(pack)
--  接收消息的拆包函数(unpack)
--  接收消息的(分发)处理函数(dispatch)
skynet.register_protocol(class)
--  返回用于延迟回应闭包，用于协程挂起一个请求，等将来时机满足，再回应的情形。非阻塞API
--  调用闭包时，第一个参数通常是 true 表示是一个正常的回应，之后的参数是需要回应的数据。
--  如果是 false ，则给请求者抛出一个异常。它的返回值表示回应的地址是否还有效。
--  如果你仅仅想知道回应地址的有效性，那么可以在第一个参数传入 "TEST" 用于检测。
skynet.response(pack)
--  发送response[raw]，非阻塞API
skynet.ret(msg, sz)
--  ret 并用 pack打包
skynet.retpack(...)
--  获取当前服务句柄
skynet.self()
--  发送消息，仅支持已注册消息，非 req-rsp 模式。非阻塞API
skynet.send(addr, typename, ...)
--  设置配置参数
skynet.setenv(key, value)
--  让框架在 ti 个单位时间后，调用 func 这个函数。
--  非阻塞 API ，当前 coroutine 会继续向下运行，而 func 将来会在新的 coroutine 中执行。
skynet.sleep(ti)
--  start_func是当前lua服务的初始化函数,也是当前服务的第一个协程的函数
--  之后在收到非response消息时dispatch_message会创建更多的协程来做逻辑
--  而调用skynet.start(start_func)的主线程会调度上述这些协程(yield)
skynet.start(start_func)
--  返回 skynet 节点进程启动的 UTC 时间，以秒为单位。 c.command("STARTTIME")
skynet.starttime()
--  ?
skynet.task(ret)
--  ?
skynet.term(service)
--  返回以秒为单位（精度为小数点后两位）的 UTC 时间。 c.command("NOW") + c.command("STARTTIME")
skynet.time()
--  相当于 sleep(0)
--  通常在想做大量的操作，又没有机会调用阻塞 API 时，可以选择调用 yield 让系统跑的更平滑。
skynet.timeout(ti, func)
--  将C指针和长度翻译成 lua 的 string
skynet.tostring
--  丢弃
skynet.trash
--  创建一个唯一的服务，调用多次service/***.lua也只启一个实例，比如clusterd和multicastd
--  uniqueservice 采用的是惰性初始化的策略。整个系统中第一次调用时，服务才会被启动起来。
--  [global为可选参数]
--  global=true时，在所有节点之间是唯一的
--  详见： https://github.com/cloudwu/skynet/wiki/UniqueService
skynet.uniqueservice(global, ...)
--  把一个C指针加长度的消息解码成一组Lua对象
skynet.unpack
--  把当前 coroutine 挂起。通常这个函数需要结合 skynet.wakeup 使用。
skynet.wait()
--  唤醒一个被 skynet.sleep 或 skynet.wait 挂起的 coroutine 。
skynet.wakeup(co)
--  将当前 coroutine 挂起 ti 个单位时间。类似wait但由底层唤醒
--  返回值会告诉你是时间到了，还是被 skynet.wakeup 唤醒 （返回 "BREAK"）。
skynet.yield()
```

2015.05.13版本把部分函数迁移到了skynet.manager模块
  
```lua
--  关闭所有服务并退出 c.command("ABORT")
skynet.abort()
--  ?
skynet.filter(f ,start_func)
--  ?
skynet.forward_type(map, start_func)
--  强制关闭指定服务 c.command("KILL",name)
--  注：skynet.kill(skynet.self()) 不完全等价于 skynet.exit() ，后者更安全。
skynet.kill(name)
--  启动一个 C 模块的服务。 command "LAUNCH"
skynet.launch(...)
--  监控服务退出
skynet.monitor(service, query)
--  为一个地址命名 c.command("NAME", name .. " " .. skynet.address(handle))
skynet.name(name, handle)
--  c.command("REG", name) 注册当前服务的名字(默认用:%x作为name，本地服务的自定义名以.打头,在32个字符以内)
skynet.register(name)
```