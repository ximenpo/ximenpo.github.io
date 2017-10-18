---
title: PHP安全配置模板
date: 2014-03-11T13:47:35+00:00
layout: post
categories:
  - 运维
tags:
  - php
  - 运维
---
<pre>;
; 位置：/etc/php.d/security.ini
;

;防止PHP信息泄漏
expose_php=Off
;display_errors=Off

;记录所有PHP错误
log_errors=On
error_log=/var/log/httpd/php_scripts_error.log

;禁止文件上传
;file_uploads=Off
;upload_max_filesize=2M

;关闭远程代码执行
allow_url_fopen=Off
allow_url_include=Off

;限制PHP访问文件系统
open_basedir=/srv/www:/tmp
session.save_path=/var/lib/php/session

;启用SQL安全模式
;sql.safe_mode=On
magic_quotes_gpc=Off

;控制POST的数据大小
;post_max_size=4M

;资源控制（DoS控制）
max_execution_time = 30
max_input_time = 30
memory_limit = 40M

;取消危险的PHP函数
cgi.force_redirect=On
disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source

;设置PHP时区
date.timezone=Asia/Shanghai

;数据获取顺序
request_order = "CGP"
</pre>