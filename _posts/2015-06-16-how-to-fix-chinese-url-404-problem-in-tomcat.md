---
layout: default
title: 解决tomcat中文URL乱码问题
date: 2015-06-16
categories: 高可用
tags: [HA, tomcat, locale]
---

## 外部表现

访问含中文的url提示404

在服务器上中文命名的文件显示为问号 ????

## 解决方案
1. 从中文显示为问号入手, 执行```locale```, 是否会报错? $LANG, $LC_CTYPE, $LC_ALL是否都有值? 都正确的话跳过此节。

    显示

    locale: Cannot set LC_CTYPE to default locale: No such file or directory

    locale: Cannot set LC_ALL to default locale: No such file or directory

1. 修改/etc/sysconfig/i18n, 增加
```
LC_CTYPE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
```

1. 生效配置
```source /etc/sysconfig/i18n```

1. tomcat配置中增加
```
    URIEncoding="utf-8"
    useBodyEncodingForURI="true"
/>
```

1. 重启tomcat服务，访问url验证
