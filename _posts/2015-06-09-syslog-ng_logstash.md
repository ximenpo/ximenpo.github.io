---
title: syslog-ng与logstash日志采集简单配置
date: 2015-06-09T15:23:11+00:00
layout: post
categories:
  - simple-bash
  - 运维
tags:
  - simple-bash
  - 运维
---
最近碰到个日志收集问题，于是接触了下相关工具，并作了个简单demo。先看任务

> 把多台游戏服务器上的日志文件[非syslog方式]汇集到一台mysql数据库。 

<!--more-->

工具都是专家推荐的，先看介绍：

> syslog-ng作为syslog的替代工具，可以完全替代syslog的服务，并且通过定义规则，实现更好的过滤功能。 

> Logstash 是一个应用程序日志、事件的传输、处理、管理和搜索的平台。你可以用它来统一对应用程序日志进行收集管理，提供 Web 接口用于查询和统计。
  
> Logstash 现在是 ElasticSearch 家族成员之一。 

跟syslog-ng类似的还有rsyslog，但没找到rsyslog对非syslog生成文件的处理，所以还是用syslog-ng了。

采用方案如下：

> １、sysnlog-ng在游戏服务器，负责收集日志，并用发送到中心日志服务器；
  
> ２、logstash在中心日志服务器，负责接收日志，并用http发送到日志处理程序；
  
> ３、nodejs/iojs建个http服务，用来接收日志，并插入mysql数据库。
> 
> ＊　事后发现，第２和３可以合并为一个处理，直接用nodejs接收tcp数据并入库。 

步骤如下：

１、安装VirtualBox，生成或复制CentOS虚拟机，并启动／登录；

２、启用simple-bash命令
  
`` source `curl https://raw.githubusercontent.com/ximenpo/simple-bash/master/use-sb.sh | bash` ``

３、安装syslog-ng
  
`sb install-syslog-ng`
  
可以先用sbc查看实际运行的install-syslog-ng脚本内容
  
```bash
[root@localhost ~]# sbc install-syslog-ng.sh
#!/bin/bash
 # require sb
 type sb 1> /dev/null 2> /dev/null || (
     echo "sb function is required"
     exit 1
 )
 # install yum repo
 yum repolist   epel    | grep epel    > /dev/null ||  sb repo-install-epel
 #install
 yum install -y syslog-ng   syslog-ng-libdbi
```

4、安装logstash
  
```sb install-logstash```
  
查看脚本执行内容
  
```bash
[root@localhost ~]# sbc install-logstash.sh
#!/bin/bash
#remove gnu version java
rpm -qa | grep 'gcj' &&  yum remove `rpm -qa | grep 'gcj'`
yum list installed  java-1.8.0-openjdk  > /dev/null  2> /dev/null ||  yum install -y java-1.8.0-openjdk
rpm -q logstash-1.5.0-1.noarch > /dev/null 2> /dev/null || rpm -i http://download.elastic.co/logstash/logstash/packages/centos/logstash-1.5.0-1.noarch.rpm
```
  
安装有点慢，请耐心等待

5、安装nvm&iojs
  
```bash
sb install-nvm
source ~/.bashrc
nvm install io.js
```

６、用sbe下载示例配置
  
```bash
sbe    syslog-ng.file-to-tcp.conf  > syslog-ng.conf
sbe    logstash.http.conf          > logstash.conf
sbe    nodejs.http.js              > http.js
```

７、执行并测试
  
需要打开３个终端，分别执行

终端１：http服务器
  
```bash
nvm use io.js
iojs http.js
```

终端２：logstash转发
  
```bash/opt/logstash/bin/logstash -f ~/logstash.conf```

终端３：生成并收集日志
  
```bashsyslog-ng -f ~/syslog-ng.conf```

然后在终端３生成日志并观察终端１的输出：
  
```bash
cd ~
echo aaaa >> act.log
echo bbbb >> act.log
```
  
此时终端１会输出一堆错误，大意是gsub在Timestamp上未定之类的。

我们可以选择把logstash.conf里的http里的format修改为json格式即可。
  
假如仍然需要用form格式，则需要手动修复这个问题，定位到_/opt/logstash/vendor/bundle/jruby/1.9/gems/logstash-output-http-0.1.4/lib/logstash/outputs/http.rb_，查找到最后encode函数，作如下修改
  
```ruby
  def encode(hash)
    return hash.collect do |key, value|
      ＃CGI.escape(key) + "=" + CGI.escape(value)
      CGI.escape(key.to_s) + "=" + CGI.escape(value.to_s)
    end.join("&")
  end # def encode
```

关闭logstash再重新打开，就可以看到nodejs的http服务显示正常了。

```bash
[root@localhost ~]# iojs http.js
Server running at http://127.0.0.1:8000/
/test
message=%3C13%3EJun++9+03%3A22%3A09+localhost.localdomain+aaa&%40version=1&%40timestamp=2015-06-09T07%3A22%3A09.660Z&host=127.0.0.1
/test
message=%3C13%3EJun++9+03%3A22%3A17+localhost.localdomain+bbb&%40version=1&%40timestamp=2015-06-09T07%3A22%3A17.670Z&host=127.0.0.1
```
