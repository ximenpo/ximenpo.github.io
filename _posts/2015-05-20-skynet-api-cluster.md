---
title: skynet API：cluster
date: 2015-05-20T00:03:26+00:00
layout: post
categories:
  - skynet
tags:
  - skynet
---
master/slave模式
  
```lua
--用来监控一个 slave 是否断开。如果slave正常将阻塞。slave断开时，会立刻返回。
harbor.link(id)
--在 salve 上监控和 master 的连接是否正常
harbor.linkmaster()
--和 harbor.link 相反。如果slave没有连接则阻塞，一直到它连上来才返回。
harbor.connect(id)
--查询全局名字或本地名字对应的服务地址。它是一个阻塞调用
harbor.queryname(name)
--注册一个全局名字。如果 handle 为空，则注册自己。skynet.name 和 skynet.register 是用其实现的。
harbor.globalname(name, handle)
```

cluster模式

1.需要在配置文件中应当配置cluster={cluster配置文件}项，配置文件项为name=ip:port形式。
  
2.cluster.call可以用lua和snax协议

<!--more-->

```lua
--调用方使用，调用name节点的service_name服务
cluster.call(name, service_name, ...)
--生成一个远程 snax 服务对象，如果给出了第三个参数 address ，那么 address 就是 snax 服务的地址，而 name 则是它的服务类型（ 绑定 snax 服务需要这个类型，具体见 snax.bind ）。
cluster.snax(node, name [,address])
--服务方调用，打开name配置项指定的tcp端口
cluster.open(name)
--调用方使用，获取name节点了解的service_name服务，之后可以普通服务句柄的方式调用
cluster.proxy(name, service_name)
--重新加载配置
cluster.reload()
```