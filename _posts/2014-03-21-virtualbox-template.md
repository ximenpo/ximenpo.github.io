---
title: 虚拟机模板及使用
date: 2014-03-21T14:12:27+00:00
layout: post
categories:
  - 运维
tags:
  - centos
  - php
  - samba
  - virtualbox
  - 虚拟机
  - 运维
---
使用virtualbox创建虚拟机模板供开发、内网使用。

<pre>==================
环境配置
==================

  名称		值		备注
  虚拟机软件	VirtualBox
  虚拟机位置	V:\	  	可使用映射网络驱动器方式

---------------
创建基础OS模板
---------------

  名称	  内容  				备注
  系统	  boot修改：硬盘->光驱,取消软盘的勾选	主板->启动顺序
  内存	  512M
  硬盘	  40G

Windows基础模板
~~~~~~~~~~~~~~~
  密码
  软件	  设备->安装增强功能
  	  C盘下新建tools目录，放置NewSID目录及软件

CentOS基础模板
~~~~~~~~~~~~~~
  密码
  配置	 网络配置
	   	vi /etc/sysconfig/network-scripts/ifcfg-eth0
 	 	HWADDR 一行删掉（修改为自动检测mac地址）
 	 	ONBOOT 修改为yes
 		MM_CONTROLLED 修改为no
  删除MAC缓存	rm /etc/udev/rules.d/70-persistent-net.rules	模板都需要
  关机 	 sync;sync;shutdown -h now

CentOS域支持模板[基于CentOS基础模板]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.	yum install samba#主机名字识别，亦可作文件共享
2.	yum install pam_krb5#活动目录密码校验
3.	authconfig-tui#配置域名及登录，配置过程参考[http://rainbird.blog.51cto.com/211214/197509]

CentOS PHP模板[基于CentOS域支持模板]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1、rpm -i http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm     #nginx yum支持

2、yum install nginx                # 安装 Nginx

3、yum install mysql mysql-server   # 安装 MySQL

4、yum install php-fpm              # 安装 PHP-FPM

5、yum install php-mysql            # 安装 PHP MySQL支持

6、yum install php-ldap             # 安装 PHP LDAP 支持

7、vi /etc/php.ini                  # 增加: open_basedir =/srv/:/tmp/

8、vi /etc/nginx/conf.d/default.conf # 修改站点及php相关支持
    location / {
        root   /srv/www;
        index  index.html index.htm index.php;
    }

    location ~ \.php$ {
        root           /srv/www;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

9、/srv 下新建目录 www# 站点主目录

10、让 Nginx/PHP-FPM/MySQL 自动启动
chkconfig --level 345 nginx on
chkconfig --level 345 php-fpm on
chkconfig --level 345 mysqld on

11、开启80端口
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
service iptables save

CentOS PHP Dev模板[基于PHP模板]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1、cp /usr/share/doc/php-common-*/php.ini-development/etc/php.ini

2、yum install php-mbstring

3、service php-fpm restart

4、修改 mySQL root 密码为 root
mysql -u root
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('mysql');

5、复制 phpMyAdmin 到 /srv/www/phpMyAdmin ，并设置所有权限为 可读 不可写 可执行

6、复制 config.sample.inc.php 为 config.inc.php

7、检查 session.save_path 是否存在，如果不存在则建立并设置相关权限[所有人可写]

8、访问 phpMyAdmin

9、rm /etc/udev/rules.d/70- -net#为虚拟机模版删除mac缓存

10、shutdown -h now  #关机

==================
生成虚拟机实例
==================

从模板复制虚拟机
~~~~~~~~~~~~~~~~~~

1.	点击虚拟机模板，在VirtualBox右上角点备份，在选择备份上按右键->复制:
2.	虚拟电脑名称设置为 "操作系统名 标志名"，点选重新初始化所有网卡的MAC地址
3.	最好选择 链接复制
4.	修改网络配置[如果需要的话]

Windows虚拟机修改
~~~~~~~~~~~~~~~~~~

执行 c:\tools\NewSID\newsid.exe，修改帐号SID以及机器名，并重启

CentOS虚拟机修改
~~~~~~~~~~~~~~~~~~

1.	vi /etc/sysconfig/network# 修改主机名
2.	修改虚拟机网络配置，然后重启reboot
3.	如果虚拟机要加入Windows活动目录域，则在重起后
	1、ifconfig    #确认机器IP为局域网内部IP，非NAT
	2、hostname    #确认机器名正确性  etc/sysconfig/network修改主机名，须重启或同时用 hostname xxx.xxx修改
	3、在DNS服务器上建主机记录，允许主机自行更新
	4、net ads join -U Administrator   #将CentOS机器加入Windows域
</pre>