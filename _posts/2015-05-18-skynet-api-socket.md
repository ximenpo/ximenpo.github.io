---
title: skynet API：skynet-socket
date: 2015-05-18T10:04:34+00:00
layout: post
categories:
  - skynet
tags:
  - skynet
---
先看下server-socket的事件：

<pre>
SOCKET_ACCEPT   仅服务端，accept成功,还需要start
SOCKET_OPEN     连接建立并处于工作状态
SOCKET_DATA     有数据
SOCKET_CLOSE    连接关闭
SOCKET_ERROR    发生错误
SOCKET_EXIT     退出事件
</pre>

<!--more-->

下面是server-socket接口：
  
<pre>
接口                            描述
socket\_server\_create        创建socket_server
socket\_server\_release       销毁socket_server
socket\_server\_poll          获取socket事件，事件类型如上所述
socket\_server\_exit          发送SOCKET_EXIT事件，表示逻辑循环退出
socket\_server\_close         主动关闭socket，会触发SOCKET_CLOSE事件
socket\_server\_listen        仅服务端，监听,socket, bind, listen
socket\_server\_start         仅服务端，启动socket，调用该函数后socket才可以接收发送数据
socket\_server\_send          发送消息
socket\_server\_connect       非阻塞的方式连接
socket\_server\_block_connect 以阻塞方式连接
socket\_server\_bind          将stdin、stdout等FD加入到管理
</pre>

skynet的socket接口对应：
  
<pre>
skynet-socket                     socket-server
SKYNET\_SOCKET\_TYPE\_DATA        SOCKET\_DATA
SKYNET\_SOCKET\_TYPE\_CLOSE       SOCKET\_CLOSE
SKYNET\_SOCKET\_TYPE\_CONNECT     SOCKET\_OPEN
SKYNET\_SOCKET\_TYPE\_ACCEPT      SOCKET\_ACCEPT
SKYNET\_SOCKET\_TYPE\_ERROR       SOCKET\_ERROR
-                                 SOCKET_EXIT
skynet\_socket\_init              socket\_server\_create
skynet\_socket\_exit              socket\_server\_exit
skynet\_socket\_free              socket\_server\_release
skynet\_socket\_poll              socket\_server\_poll
skynet\_socket\_send              socket\_server\_send
skynet\_socket\_send\_lowpriority socket\_server\_send\_lowpriority
skynet\_socket\_listen            socket\_server\_listen
skynet\_socket\_connect           socket\_server\_connect
skynet\_socket\_block\_connect    socket\_server\_block\_connect
skynet\_socket\_bind              socket\_server\_bind
skynet\_socket\_close             socket\_server\_close
skynet\_socket\_start             socket\_server\_start
skynet\_socket\_nodelay           -
skynet\_socket\_udp               -
skynet\_socket\_udp_connect       -
skynet\_socket\_udp_send          -
skynet\_socket\_udp_address       -
</pre>