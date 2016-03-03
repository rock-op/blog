---
layout: default
title: 使用Jmeter的一些点
date: 2016-03-03
categories: QA
tags: [HA, Jmeter]
---
## Install
```bash
brew update
brew install jmeter --with-plugins
```

## Using External package/classes
```
cp irs.jar bcprov-jdk15-144.jar json-lib-2.2.2-jdk15.jar commons-lang-2.4.jar ${JMETER_HOME}/libexec/lib/
```
or

`modify bin/user.properties, user.classpath`

and then restart jmeter

## Debug
查看更详细的错误日志

```
JMeter directory -> bin -> jmeter.properties -> change the 'log_level.jmeter=INFO' to 'log_level.jmeter=DEBUG'
```

## Error
执行`jmeter`出错，`No X11 DISPLAY variable was set, but this program performed an operation which requires it.`
解决方案，

`bin/jmeter -n -t test.jmx -l logs/test.jtl`

```
-n [This specifies JMeter is to run in non-gui mode]
-t  [name of JMX file that contains the Test Plan]
-l  [name of JTL file to log sample results to]
-j  [name of JMeter run log file].
Besides these options, JMeter has several other parameters that can be used for running in non-GUI mode.
-R [list of remote servers] Run the test in the specified remote servers
-H [proxy server hostname or ip address]
-P [proxy server port]
These options are used for remote execution of JMeter tests and for using JMeter through a proxy server.
```
