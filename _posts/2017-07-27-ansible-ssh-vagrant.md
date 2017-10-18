---
title: Ansible/SSH直连Vagrant
date: 2017-07-27T17:13:36+00:00
layout: post
categories:
  - 运维
  - 问题
tags:
  - 问题
---
使用vagrant创建虚拟机后，可以用<q>vargrant ssh</q>直接ssh上去，但是如何直接用ssh或者ansible进行连接呢？

1、我们先用<q>vargrant ssh-config</q>生成ssh配置文件，然后再用ssh/ansible的<q>-F</q>参数指定这个配置文件即可：
  
```bash
# 生成配置
vagrant ssh-config > ../ssh_config
# 用ssh直连
ssh -F ../ssh_config core-01
# 用ansible直连
ansible -i 'core-01,' --ssh-extra-args='-F ../ssh_config'  all -m ping
```

2、（仅支持ansible）

若vagrant使用了ansible的provision，那么Vagrant会基于 `Vangrantfile` 自动为我们生成 `Ansible` 的`inventory` 文件，并放在与 `Vgrantfile` 文件同级的 `.vagrant/provisioners/ansible/inventory/vagrant\_ansible\_inventory` 文件中。 使用ansible时只要用 `-i` 参数指定下这个 `inventory` 文件即可。
