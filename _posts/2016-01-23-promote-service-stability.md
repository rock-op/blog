---
layout: default
title: 服务稳定性提升工作
date: 2016-01-23
categories: 高可用
tags: [HA, stability]
---

# 服务稳定性提升工作

**最重要的不是发现，而是执行!**

## 现有问题的解决
1. 清理脏数据
1. 重构uploadOrderinfo接口异步化
1. 解决现有exception日志
1. 解决数据库从库IOPS报警问题
1. 解决常态下的账单重复问题
1. 优化慢SQL

## 防患于未然
1. 数据库层面优化
   - 索引合理性review
   - 监控巡查，IOPS, 连接数
   - 慢SQL优化落地
   - 拆表
1. 服务层面优化
   - 监控
      - 补充各模块exception监控
      - 补全nginx监控
      - 补全redis监控
   - 部署
      - 拆分deploy.yml
      - 部署模式优化(小流量->全流量)
   - 日志规范(接口、返回码、耗时)
   - 关联模块
      - redis迁移至阿里云集群
      - activemq集群化
   
## 容量提升
1. 压测
