---
layout: default
title: 初创公司的配置工具选型
date: 2015-06-11
categories: 高可用运维
tags: [ansible, op-tools]
---

# 初创公司的配置管理工具选型

* 生产环境: 阿里云
* 公司规模: 创业公司
* 工具选择Ansible, 原因有以下几点：
    1. 目前开源的配置管理工具有puppuet, chef, salty, ansible, 其中puppet, chef使用ruby开发，需要ruby环境。salty, ansible采用python开发。由于所有服务器默认都有python环境，采用ansible 需要额外的依赖更少一些。
    1. ansible基于ssh协议，无需安装任何agent
    1. 可作为批量执行工具
    1. 支持使用配置文件playbook描述环境/执行命令, 便于扩展(后面会说到我们是如何使用ansible做持续部署工具的)

    对于1和2，深有感触，之前使用puppet真是要疼死，新机器到货之后，不仅要在所有机器上安装puppet，还TMD要安装ruby啊魂淡！想想要是到货100台机器，装到最后，算了，我还是直接ssh过去手动搞下比较方便...

* 管理机器: 目前是通过静态方式将所有host ip写入/etc/ansible/hosts文件中，做好分组。暂时规模太小，先静态维护就够了。
