---
title: SVN服务器配置
date: 2014-06-11T13:58:53+00:00
layout: post
categories:
  - 运维
tags:
  - SVN
  - 运维
---
SVN管理采用iF.SVNAdmin。

<pre>SVN服务器配置

1、从 CentOS6.5_Samba 复制虚拟机

2、安装必要软件

yum install httpd httpd-devel subversion mod_dav_svn
确认安装成功：
svn --version

3、配置apache选项

vi /etc/httpd/conf/httpd.conf
KeepAlive On #从Off修改为On
DocumentRoot "/srv"#修改为网站根目录
ServerAdmin root@localhost	#修改为管理员邮箱
#ServerName www.example.com:80	#可改可不改
#在httpd.conf最后增加svn虚拟目录配置
&lt;location /svn>
	DAV svn
	SVNParentPath		/srv/svn
	AuthzSVNAccessFile	/srv/svn_auth/svn_auth.conf
	AuthType Basic
	AuthName "SVN"
	AuthBasicProvider ldap
	AuthLDAPBindDN		"ldap@xxx.xxx"
	AuthLDAPBindPassword	xxxxxxxx
	AuthLDAPURL		"ldap://dc1.xxx.xxx:389/CN=Users,DC=xxx,DC=xxx?sAMAccountName?sub?(objectClass=*)"
	Allow			from All
	require			valid-user
&lt;/location>

4、在/srv下创建 /srv/svn  /srv/svn_auth 目录，并创建 svn_auth.conf 文件

mkdir /srv/www#网站主目录
mkdir /srv/svn_repo#svn主目录
mkdir /srv/svn_auth#svn授权目录
touch /srv/svn_auth/svn_auth.conf#svn授权配置文件
chmod -R 777 /srv
chcon -R -h -t httpd_sys_content_t /srv#修改selinux对/srv的配置

5、启动apache，并配置为自动启动
service httpd start
chkconfig httpd on

6、开放80端口
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
service iptables save

7、安装php[为iF.SVNAdminr安装作准备]
yum install php php-mbstring php-ldap

8、将iF.SVNAdminr解压缩到/srv/www，访问该http地址并修改配置如下：
SVNAuthFile=/srv/svn_auth/svn_auth.conf

UserViewProviderType=ldap
UserEditProviderType=
GroupViewProviderType=svnauthfile
GroupEditProviderType=svnauthfile
RepositoryViewProviderType=svnclient
RepositoryEditProviderType=svnclient

SVNParentPath=/srv/svn_repo
SvnExecutable=/usr/bin/svn
SvnAdminExecutable=/usr/bin/svnadmin
HostAddress=ldap://dc1.xxx.xxx:389/
ProtocolVersion=3
BindDN=CN=ldap,CN=Users,DC=xxx,DC=xxx
BindPassword=xxxxxxxx

BaseDN=CN=Users,DC=xxx,DC=xxx
SearchFilter=(objectClass=user)
Attributes=sAMAccountName

9、登录iF.SVNAdmin并进行SVN相关管理
</pre>