---
title: MySQL异常关闭导致无法启动的处理一例
date: 2014-03-11T14:02:43+00:00
layout: post
categories:
  - 运维
tags:
  - mysql
  - 运维
---
原因是sock文件没被清理掉。

<pre>在 /etc/rc.d/rc.local 里添加如下代码：
# clean mysqld temp socket.
service mysqld status | grep stopped > /dev/null && if [ -e /var/lib/mysql/mysql.sock ]; then
	rm /var/lib/mysql/mysql.sock
	service mysqld start
fi
</pre>