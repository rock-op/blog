---
layout: default
title: 故障追查，分析java heap dump
date: 2016-02-16
categories: 高可用
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

<img src="https://github.com/rock-op/blog/blob/gh-pages/_images/Eclipse_Memory_Analyzer_hprof.jpg?raw=true" width="500" height="300" alt="dominator_tree"/>

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

EOFException什么意思呢，从官方文档上来看，当写入时，意外地到达了文件/流的末尾。

> Signals that an end of file or end of stream has been reached unexpectedly during input.
> This exception is mainly used by data input streams to signal end of stream. Note that many other input operations return a special value on end of stream rather than throwing an exception.

是抛出的Exception，但为什么Exception Logs没有打到日志文件中，反而存在内存的LinkedList中呢? 进入日志目录发现，服务日志的权限被改成了root，使用work启动的服务写不进去。

所以完整的原因应该是，

1. 服务出现大量Exception
1. 正常逻辑是Exception信息存储于LinkedList，刷到磁盘之后从LinkedList中删掉
1. 但因为服务日志权限问题，异常日志无法写入磁盘，一直堆积在内存中，无法被GC掉。

但这个EOFException在重启之后就没有了，怀疑是在Eden区域没有足够内存，导致有用户请求过来时，请求的内容被截断了。

另外，从GC日志中发现，从2.4就已经有频繁的Full GC了，每分钟有3次Full GC。

将日志权限改回work，重启服务，检查日志滚动已经恢复正常。针对于这个EOFException，后续再观察。
