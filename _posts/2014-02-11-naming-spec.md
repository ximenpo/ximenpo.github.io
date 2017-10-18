---
title: 用户及计算机命名参考
date: 2014-02-11T13:52:54+00:00
layout: post
categories:
  - 运维
tags:
  - 规范
  - 运维
---
供参考。

<pre>一、用户名：

用户名使用“姓+名字缩写”的方式。如zhaojianwei，可以使用“zhaojw”，如果出现重名的情况，则添加工号后缀以区分，如“zhaojw15”。

二、工作计算机

工作计算机使用 “PC-用户名”的方式，比如“zhaojw”，则对应工作计算机名称为“pc-zhaojw”，如果该用户有多台计算机的，再添加后缀区分，如“pc-zhaojw-1”。

三、服务器

服务器直接按"功能前缀+序号"方式命名，序号不是必须的。比如：

 前缀	 类别		 	 示例 	  	 备注
 vhost	 物理机，用来放置虚拟机	 vhost1
 dc	 域控服务器	  	 dc1
 oa	 办公自动化服务器	 oa
 share	 共享资源	 	 share

四、特殊用户

帐号	密码  	显示名   说明
ldap		LDAP  	 ldap查询帐户
keeper 	  	管理员   ldap查询管理员

</pre>