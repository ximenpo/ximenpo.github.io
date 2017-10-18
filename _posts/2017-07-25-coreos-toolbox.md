---
title: CoreOS的toolbox相关
date: 2017-07-25T15:43:27+00:00
layout: post
categories:
  - 运维
  - 问题
tags:
  - 问题
---
CoreOS上有个很好用的工具叫toolbox，默认相当于一个与主机环境绑定的fedora:latest容器环境（是systemd的容器，非Docker）；这是个脚本文件，代码不长，但内容不少；）

下面是我碰到的一些情况及处理方案：

### 自定义镜像或其他操作

看代码可以发现，toolbox执行时会在主要变量设置后去检测／执行以下两个文件（如果存在的话）
  
```bash
source     /etc/default/toolbox
source     "${HOME}"/.toolboxrc
```
  
<!--more-->

所以解决方案就很简单了，随便在上面哪个文件里设置对应的变量或相关操作即可，比如：
  
```bash
TOOLBOX_DOCKER_IMAGE=centos
TOOLBOX_DOCKER_TAG=7
```

### 从指定的url下载镜像

看代码也是可以的，只要设置TOOLBOX\_DOCKER\_ARCHIVE变量即可（指向一个.tar.xz格式的镜像文件），但代码里感觉处理得不是很完善，所以在自定义配置文件里要多加些代码：
  
```bash
#清空镜像名／TAG至少一个
TOOLBOX_DOCKER_IMAGE=fedora
TOOLBOX_DOCKER_TAG=
#设置TOOLBOX_NAME名
TOOLBOX_NAME=fedlra-latest
#设置镜像下载地址(须wget支持)
TOOLBOX_DOCKER_ARCHIVE=http://xxxxxxxxx/xxx/xxx.tar.xz
```

### 镜像可不可以用本地文件或者手动安装&#8230;

在前一步中，由于脚本里是用wget下载的，而wget不支持本地文件（也不支持file://），所以只能手动处理了：

```bash
# 代码复制并修改自toolbox，可放入.toolboxrc执行
# - X_IMG_NAME为docker镜像名
# - X_IMG_TAG为docker镜像标记
# - 其他变量同toolbox
machinename=$(echo "${USER}-${X_IMG_NAME}-${X_IMG_TAG}" | sed -r 's/[^a-zA-Z0-9_.-]/_/g')
machinepath="${TOOLBOX_DIRECTORY}/${machinename}"
osrelease="${machinepath}/etc/os-release"
if [ ! -f "${osrelease}" ]; then
    tmpdir=$(mktemp -d -p /var/tmp/)
    tmpfile=${tmpdir}/docker-${X_IMG_NAME}-${X_IMG_TAG}.tar
    trap "sudo rm -rf ${tmpdir}" EXIT PIPE
    docker pull "${X_IMG_NAME}:${X_IMG_TAG}" || exit    1
    docker save "${X_IMG_NAME}:${X_IMG_TAG}" -o "${tmpfile}" || exit    1
    sudo tar -C ${tmpdir} -xf "${tmpfile}"
    sudo mkdir -p ${machinepath}
    sudo find ${tmpdir} -name layer.tar -type f -exec tar -C ${machinepath} -xf {} \;
    trap - EXIT PIPE
    sudo rm -rf ${tmpdir}
fi
```

### \*其他方案\*

toolbox其实也只是一个脚本而已，自己复制一份按自己需求修改下，然后运行自己的toolbox即可；）

---
  
备注：以上功能，我已经写了一个专门的ansible的role模块coreos-toolboxrc（https://github.com/ximenpo/ansible-role-coreos-toolboxrc），可以参考使用。