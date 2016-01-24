---
layout: default
title: 系统nproc参数过小导致Resource temporarily unavailable 
date: 2015-10-19
categories: 高可用架构
tags: [nproc, HA]
---

默认阿里云ECS, 非root账户打开的最多processes数为1024。

`$ ulimit -a`

`max user processes              (-u) 1024`

对于JAVA进程而言，这里的process是算JAVA的进程数还是算JAVA线程数? 对此进行了验证，结论是‘线程数’，验证过程：

`cat /etc/security/limits.d/90-nproc.conf`

`*          soft    nproc     1024`

把此值修改为一个比较小的值，比如50，之后`su - work`，重启下用work启动的服务进程。
如果`pstree | grep java`看到的值超过50之后，会提示`Resource temporarily unavailable`.

对于这个系统参数的修改，可以在新机器到货时，把更改的动作放到`ansbile-playbook`的初始化任务中。