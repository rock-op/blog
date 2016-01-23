---
layout: default
title: 新模块上线准入流程 
date: 2015-08-27
categories: 持续交付
tags: [HA, CI]
---
# 新模块上线准入流程

## 基本概念：

- 开发环境：RD自己电脑
- 测试环境：内网服务器，172.16.x.y
- 预上线环境：阿里云服务器，公司内部访问的“线上”环境
- 生产环境/线上：阿里云服务器，真实对外提供服务的环境
- jenkins：编译、打包、部署的平台

## 模块如果要部署到测试环境/生产环境，

- 请确保模块的SVN根目录下存在build.sh，用于编译打包。见下方的【编译打包】
- 请确保SVN中存在多套配置文件，适用于不同的部署环境。见下方的【配置管理】
- 请确保SVN中存在启停脚本，已有启停脚本模板，请复用。见下方的【启停脚本】
- SVN中服务端口约定：tomcat类端口统一为8080，server类端口统一为9090。方便监控添加。
- 在内网jenkins机器上建立编译打包任务，拷贝其他模块的任务即可，任务名为模块名.build
- 在生产环境jenkins.passiontec.cn建立模块的部署任务，拷贝其他模块的任务即可，任务名为 环境名.模块名.deploy

### 编译打包

- 不管是采用ant、maven还是gradle，都需要提供build.sh, 屏蔽编译细节。
- sh build.sh 通过参数dev/test/preonline/online 指定要使用哪个环境的配置
- 执行完毕后，在build.sh同级目录下生成output目录，
    tomcat的工程, 会自动把target/*.war包在output目录下
    独立的java工程，会自动把target/{bin,conf,lib}放在在output目录下
- build.sh 有模板，tomcat类服务请使用irs-web下的build.sh，server服务请使用om-server下的build.sh。

### 配置管理

一般而言，模块需要部署在4套环境下，

- dev: rd本地，自己开发机
- test: 测试环境，内网服务器
- preonline: 预上线，阿里云ECS（可选）
- online: 生产环境，阿里云ECS

对应有4套配置文件，分别对应4个目录，每个目录中配置都是全量的。执行build.sh + 部署的环境名，打出不同配置的包。

### 启停脚本

启停脚本无需自己编写，已有模板，兼容Tomcat类web工程及独立的java服务，请尽情使用。请拷贝任意模块的bin目录到自己的模块SVN中。bin中的脚本作用说明如下

- run.sh  # 无需修改，使用方式: sh run.sh stop/start
- java_env.sh  # java的一些参数，指定jdk版本与模块的Jvm大小
- tomcat_env.sh  # tomcat相关参数设定，对于独立的java服务，请将此文件删除
- webserver_env.sh # 模块个性化设置，请修改main_class
- preonline_modify.sh  # 预上线部署时修改部分配置，如果有预上线需求，请修改此文件
- log_manager.sh  # 日志清理脚本，可修改日志保存时间

### 目录结构

Tomcat容器: 

`ls -l`

>/home/work/jdk-1.7
>
>/home/work/apache-tomcat-7.0.${MODULE_NAME}
>
>/home/work/${MODULE_NAME}
>
>--- bin     # 启停脚本目录
>
>--- logs -> /home/work/apache-tomcat-7.0.${MODULE_NAME}/logs
>
>--- WEB-INF
>
>--- META-INF
>
>--- ...

独立的JAVA服务:

`ls -l`

> /home/work/jdk-1.7
>
> /home/work/${MODULE_NAME}
>
> --- bin   # 启停脚本目录
>
> --- conf  #  配置文件与class文件
>
> --- lib   # 依赖的各种jar包
>
> --- logs -> /home/work/var/${MODULE_NAME}/logs
