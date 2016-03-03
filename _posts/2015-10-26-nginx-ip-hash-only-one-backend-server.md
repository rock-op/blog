---
layout: default
title: nginx ip_hash转发至一台机器
date: 2015-10-26
categories: 高可用
tags: [nginx, HA, stability]
---

## 问题描述
阿里云nginx ip_hash将所有请求转发到了一台机器，即使upstream后端配置了多台tomcat server

## 排查过程
查看nginx的error.log, 发现大部分的client ip是来自于10.aaa.bbb网段

> 2015/10/25 12:01:03 [warn] 21580#0: *12412231 an upstream response is buffered to a temporary file /home/work/nginx/proxy_temp/2/55/0000075552 while reading upstream, client: 10.aaa.bbb.ccc, server: ...

怀疑nginx未获取到真实的用户ip，而获取到上游SLB集群的IP。

nginx官方网站中关于ip_hash的说明

> Specifies that a group should use a load balancing method where requests are distributed between servers based on client IP addresses. The first three octets of the client IPv4 address, or the entire IPv6 address, are used as a hashing key. The method ensures that requests from the same client will always be passed to the same server except when this server is unavailable. In the latter case client requests will be passed to another server. Most probably, it will always be the same server as well.

意思是，对于IPv4，ip_hash会使用IP的前3部分作为哈希的Key，一个SLB集群对应的网段都是一致的，导致ip_hash之后的请求全部落到一台upstream机器。

## 解决方案
让nginx获取到用户的真实IP，使用real client ip做ip_hash

1. 重新编译nginx, 增加--with-http_realip_module模块
1. 在nginx.conf中http{}中，增加
```
real_ip_header X-Forwarded-For;
set_real_ip_from 0.0.0.0/0;
real_ip_recursive on;
```
1. 重启nginx，查看error.log检查client: 对应日志，已经变成了用户真实的外网出口IP
1. 检查upstream的机器流量，已趋于均衡
