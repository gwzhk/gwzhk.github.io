---
title: Nginx的日志及日志切割
date: 2018-09-21 22:49:17
categories:
- nginx
tags: 
- nginx
- 日志
---

> Nginx是我们的各个系统的入口，承担着静态服务器、反向代理服务器、负载均衡服务器的任务。Nginx的日志可以让我们知道访问的来源、使用的终端、客户端ip等信息，也能够发现某个服务或系统的性能瓶颈。Nginx的日志需要按照一定的格式记录，方便我们查询信息、分析问题。所以，我们需要稍微了解下Nginx的日志格式以及日志切割。

<!-- more -->

# Nginx的日志

> error_log：通过错误日志，你可以得到系统某个服务或server的性能瓶颈等；
> access.log：通过访问日志，你可以得到用户地域来源、跳转来源、使用终端、某个URL访问量等相关信息。



## Nginx日志的配置

nginx的日志格式一般在http模块中定义，然后在各个server模块中引用该日志格式。日志默认存放在nginx的logs目录下。

```nginx
#定义日志的格式
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

#日志生成的到Nginx根目录logs/access.log文件，默认使用“main”日志格式，也可以自定义格式
access_log logs/access.log main
```

## Nginx日志的参数

nginx日志参数明细见下表：

| 参数                  | 含义                                                 |
| --------------------- | ---------------------------------------------------- |
| $remote_addr          | 客户端的ip地址(代理服务器，显示代理服务ip)           |
| $remote_user          | 用于记录远程客户端的用户名称（一般为“-”）            |
| $time_local           | 用于记录访问时间和时区                               |
| $request              | 用于记录请求的url以及请求方法                        |
| $status               | 响应状态码，例如：200成功、404页面找不到等。         |
| $body_bytes_sent      | 给客户端发送的文件主体内容字节数                     |
| $http_user_agent      | 用户所使用的代理（一般为浏览器）                     |
| $http_x_forwarded_for | 可以记录客户端IP，通过代理服务器来记录客户端的ip地址 |
| $http_referer         | 可以记录用户是从哪个链接访问过来的                   |

## Nginx日志的切割

```bash
#开启系统日志，如不开启，看不到定时任务日志
/etc/init.d/rsyslog start
#开启定时任务
/etc/rc.d/init.d/crond start
```

编写日志切割的shell脚本：

```bash
#!bin/bash
#设置日志存放的目录
LOG_PATH=/usr/local/webserver/nginx/logs/
#备份文件名称
LOG_SUFFIX=$(date -d "yesterday" +%Y-%m-%d-%H%M)
#重命名日志文件
mv ${LOG_PATH}/access.log ${LOG_PATH}/access_${LOG_SUFFIX}.log
mv ${LOG_PATH}/error.log ${LOG_PATH}/error_${LOG_SUFFIX}.log
#向Nginx主进程发送USR1信号。USR1信号是重新打开日志文件
kill -USR1 $(cat /usr/local/webserver/nginx/nginx.pid)
```

给脚本文件赋予执行权限：

`chmod +x logscut.sh`

```ba&#39;sh
#配置用户的定时任务
crontab -e
*/1 * * * * /usr/local/webserver/nginx/sbin/logscut.sh
```



linux下的定时任务详解请点击[这里](https://www.cnblogs.com/zoulongbin/p/6187238.html)。