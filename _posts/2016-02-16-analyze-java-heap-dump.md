---
layout: default
title: 故障追查，分析java heap dump
date: 2016-02-16
categories: 高可用架构
tags: [java, HA, stability, Memory Analyzer]
---

2016.02.15 15:19 收到一台irs-iface机器的crash报警，随即恢复(zabbix有自动重启机制)。

但自动恢复了，就算万事大吉了吗? crash的原因是什么? 会不会对服务稳定性有影响/风险? 这种深究的精神是每个工程师都需要具备的品质。

排查重启前的catalina.out，发现

```
java.lang.OutOfMemoryError: GC overhead limit exceeded
Dumping heap to java_pid18124.hprof ...
Heap dump file created [1082759390 bytes in 22.848 secs]
```

很显然，服务`OutOfMemory`了，在挂掉之前出现频繁`Full GC`, 因为jvm启动参数增加了`-XX:+HeapDumpOnOutOfMemoryError`，OOM之后自动做了HeapDump.

用MemoryAnalyzer分析生成的heapDump文件，使用dominator_tree, 点开之后

![dominator_tree](https://github.com/rock-op/blog/blob/gh-pages/_images/Eclipse_Memory_Analyzer_hprof.jpg?raw=true)

发现是服务创建了9万多个LinkedList对象，每个对象大小有10K，将近1个G。

点开看下其中存的是什么内容


```
java.io.EOFException
    at ---.+++.utils.memo.util.AccessOut.readInt(AccessOut.java:621)
    at ---.+++.***.utils.mem.util.Saveable.readArray(Saveable.java:284)
    ...
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:745)
```

是抛出的Exception，但为什么Exception Logs没有打到日志文件中，反而存在内存的LinkedList中呢? 进入日志目录发现，服务日志的权限被改成了root，使用work启动的服务写不进去，就一直堆积在内存中，并且无法被GC掉。

将日志权限改回work，重启服务，检查日志滚动已经恢复正常。
