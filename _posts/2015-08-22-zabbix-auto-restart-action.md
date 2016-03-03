---
layout: default
title:  zabbix实践：服务异常后自动执行预案 
date: 2015-08-22
categories: 高可用
tags: [zabbix, HA, stability]
---

#  zabbix实践：服务异常后自动执行预案 

## 题记
当服务异常之后，不应该只是发短信/邮件通知op，之后再由op登陆vpn, 登陆服务器处理。整个流程应该再顺畅些，检测到服务异常 -> 触发预案执行。单纯执行预案的op是没有存在价值的。不过，加那么多无用的报警来骚扰op也是耍流氓啊亲。

ok，再考虑一个问题，我们要怎么检测到服务是真的异常了? 通过什么监控指标?
其实我们在说服务异常时，通常是指两个方面，

1. 服务挂了，请求拒绝。
2. 服务没挂，但性能变差，体验变差。

先来考虑第一个方面，也就是服务挂了我们要能监控到，这是最基本的要求。通过什么指标来监控服务的存活性呢?

**方案1.** 端口存活性监控，端口不在就是挂了。问: 有没有可能端口还在，服务挂了?

**方案2.** 流量监控，没流量了就是挂了。问: 有没有可能低峰期(比如凌晨)就是没流量，但服务是好的?

**方案3.** java线程数监控(我们的服务是java的)，线程数超过了某个值就是挂了。问: 超过了多少合适，没超过就一定没问题吗?

**方案4.** 真正发起一个请求，如果没返回/返回异常就表示挂了。问: 发起什么请求, 有没有可能这个类型的请求没问题，那个请求就有问题?

这么对比下来，我们决定采用方案4作为服务存活性的监控方案。毕竟，没有什么监控可以覆盖100%的场景。

我们的服务分为两类，

1. web类服务，对外暴露http的接口；可以定期curl一个监控文件，看http return code.
1. server类服务，为web服务提供数据/逻辑支持，走rpc的调用。对于rpc的服务，暂时没想到特别好的方案，暂时通过尝试connect，看能不能连上。

再考虑一个问题，服务挂了之后的预案是什么?

服务挂了，最简单粗暴的方式就是重启服务。当然，需要在重启前保存现场。

重启的脚本最好放在服务本身的控制脚本中，便于维护。不要额外单独维护一个“预案脚本”。

## 实践

把监控脚本分发到所有的服务器(zabbix_agentd)上。

配置templates的item，每分钟检测一次。比如item叫serviceA.is_alive，正常为1，异常为0。

配置templates的trigger，连续两分钟异常则触发trigger, trigger的表达式应该这么写

`({Template serviceA:serviceA.is_alive.last(#2)}<>1 and {Template serviceA:serviceA.is_alive.last()}<>1 ) or {Template serviceA:serviceA.is_alive.nodata(120)}=1`

为了方便看的话，可以为item添加一个graph。
之后，配置Actions，建议:

1. Default Message: zabbix默认的default message太长了，精简如下
`[{TRIGGER.STATUS}][{TRIGGER.SEVERITY}][{EVENT.TIME}][{TRIGGER.NAME}][{HOST.IP1}][{ITEM.NAME1} ({HOST.NAME1}:{ITEM.KEY1}): {ITEM.VALUE1}]`

2. 勾选上Recovery Message，恢复正常时给你再发一条消息
3. `Condition: (A and B)`

   `A. Trigger value = Problem`
   `B. Trigger = Template serviceA: trigger_name`
   
4. Operations: 

   `operation type: Remote Command`
   
   `target: Current Host`
   
   `type: custom script`
   
   `execute on: zabbix agent`
   
   ```
   // zabbix_agentd需要是root启动，以work账号执行重启的命令。
   // restart逻辑集成在服务自身的启停脚本中。
   commands: su - work -c "bash /home/work/serviceA/bin/run.sh restart"
   ```

另外，zabbix也提供`web scenarios`的监控方式，去get一个url看结果/返回码。

在判断服务存活性的场景里，web scenarios是不合适的。因为web scenarios是由zabbix_server发起的。

zabbix_server异常时，web scenarios的执行结果可能会异常。同时也不方便在action中指定要执行命令的host.
