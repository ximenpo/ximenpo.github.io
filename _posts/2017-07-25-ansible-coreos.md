---
title: Ansible与CoreOS
date: 2017-07-25T15:08:08+00:00
layout: post
categories:
  - 运维
  - 问题
tags:
  - 问题
---
最近开始关注Ansible与CoreOS的操作，在网上搜索了一圈，主要发现如下几个文章和参考代码：

  * <a href="https://www.tazj.in/en/1410951452" target="_blank" rel="noopener">Provisioning CoreOS with Ansible: Basic Setup</a>
  * <a href="http://python.jobbole.com/84255/" target="_blank" rel="noopener">使用 Ansible 管理 CoreOS</a>

  * <a href="https://github.com/defunctzombie/coreos-ansible-example" target="_blank" rel="noopener">defunctzombie / coreos-ansible-example</a>
  * <a href="https://github.com/tazjin/coreos-ansible" target="_blank" rel="noopener">tazjin / coreos-ansible</a>
  * <a href="https://github.com/defunctzombie/ansible-coreos-bootstrap" target="_blank" rel="noopener">defunctzombie / ansible-coreos-bootstrap</a>

研读总结后，其要点如下：

  1. Ansible操作CoreOS时的Python运行环境
  2. 文件相关的同步及部署
  3. cloudinit相关的user_data配置文件生成及部署
  4. systemd/fleet/docker相关Unit/Service等的配置及状态

<!--more-->

下面就挨个解释一下：

### CoreOS端Python运行环境配置

上面相关的两篇文章分别采用了两种不同的配置方式：

  1. 直接下载pypy运行环境到~/bin目录，并设置ansible的python解释器变量为 
    ```"PATH=~/bin:$PATH python"```

  2. 采用CoreOS自带的toolbox方式，使用toolbox里边的python环境，并设置ansible的python解释器变量为（原文是设置了/opt/bin/python这个脚本） 
    ```toolbox --bind=/home:/home python```

这两种方式在最新的ansible版本上都有问题（2.3.1），主要是创建子进程时取的程序名不对，这里我采取的对策是：
  
```bash
1. python脚本部署到 /opt/bin/ 目录下（该目录缺省就在CoreOS登录后的PATH中）
2. 给play设置environment的PATH变量，增加/opt/bin目录；
3. 设置ansible_python_interpreter变量为python，而非默认的/usr/bin/python
```

另外需要注意的是，在最新的fedora:latest镜像中，默认安装的是python3，所以ansible\_python\_interpreter的值要设置为python3，同时/opt/bin/python也要修改为/opt/bin/python3。

```
此处有坑：
由于toolbox脚本采用rkt进行fetch相关的docker容器，而fetch时是直接访问自己的url进行下载，所以你懂的，伟大的墙，时好时坏，时快时慢，这个问题以及toolbox相关坑我统一总结到另外一篇。
```

### 文件相关的同步及部署

采用toolbox方式时，文件同步的CoreOS端，要添加/media/root前缀，原因是toolbox脚本执行时映射系统目录到/media/root下了，具体可参考其源码（很短，很精悍）。

### cloudinit相关的user_data配置文件生成及部署

同步／部署方式同上，目标位置是[/media/root]/var/lib/coreos-install/user_data

### systemd/fleet/docker相关Unit/Service等的配置及状态

这个貌似没什么特别的，就不专门写了。