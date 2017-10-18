---
title: ansible-tower-cli无法连接Tower站https
date: 2017-07-12T11:37:14+00:00
layout: post
categories:
  - 运维
  - 问题
tags:
  - 问题
---
昨天在MAC上测试ansible-tower-cli时，碰到无法连接https的Tower站，错误信息如下：
  
<!--more-->

```bash
(python27) world:python27 simple$ tower-cli user list -vvvv --description-on
*** DETAILS: Getting records. *************************************************
GET http://10.42.0.42/api/v1/users/
Params: {'page': 1}
WARNING:urllib3.connectionpool:Retrying (Retry(total=2, connect=None, read=None, redirect=0, status=None)) after connection broken by 'ProtocolError('Connection aborted.', error(54, 'Connection reset by peer'))': /api/v1/users/?page=1
WARNING:urllib3.connectionpool:Retrying (Retry(total=1, connect=None, read=None, redirect=0, status=None)) after connection broken by 'ProtocolError('Connection aborted.', error(54, 'Connection reset by peer'))': /api/v1/users/?page=1
WARNING:urllib3.connectionpool:Retrying (Retry(total=0, connect=None, read=None, redirect=0, status=None)) after connection broken by 'ProtocolError('Connection aborted.', error(54, 'Connection reset by peer'))': /api/v1/users/?page=1
Cannot connect to Tower:
HTTPSConnectionPool(host='10.42.0.42', port=443): Max retries exceeded with url: /api/v1/users/?page=1 (Caused by ProtocolError('Connection aborted.', error(54, 'Connection reset by peer')))
Error: There was a network error of some kind trying to connect to Tower.
The most common reason for this is a settings issue; is your "host" value in `tower-cli config` correct?
 Right now it is: "http://10.42.0.42".
```
  
尝试了升级openssl后错误依旧，最后通过 <https://stackoverflow.com/questions/27114094/requests-mechanize-urllib-fails-but-curl-works> 发现只要安装／升级下 pyOpenSSL 即可：

```
(python27) world:python27 simple$ pip install pyOpenSSL
```

然后就可以顺利地取到数据了：
  
```
(python27) world:python27 simple$ tower-cli user list -vvvv --description-on
 *** DETAILS: Getting records. *************************************************
 GET http://10.42.0.42/api/v1/users/
 Params: {'page': 1}
 == ======== ================= ========== ========= ============ =================
 id username email             first_name last_name is_superuser is_system_auditor
 == ======== ================= ========== ========= ============ =================
 1  admin    admin@example.com                      true         false
 == ======== ================= ========== ========= ============ =================
 (python27) world:python27 simple$
 ```