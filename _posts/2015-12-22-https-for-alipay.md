---
layout: default
title: 支付宝支持的开发者SSL证书 
date: 2015-12-22
categories: 高可用
tags: [https, alipay]
---

*公司的支付平台要对接支付宝和微信，在支付宝上折腾了两天，记录于此。*

支付宝开发者文档要求:
 
>如果是https，那么需要安装ssl证书，证书要求有如下两点：
>
>1. 要求“根证书缺省内置在JDK 1.6的信任根证书库中”，如果不理解请咨询证书供应商，提出：要求SSL证书由JDK?1.6.0_21缺省内置的根CA签发。
>1. 只支持官方机构颁发的正版SSL证书，不支持自签名。

这个里边说了两个限制：

1. 不允许是自制ssl证书；
1. 很重要，但常被忽略，『根证书缺省内置在JDK 1.6的信任根证书库中。』

这句话比较拗口，我来解释下它的意思，

`根证书`: root certificate, 是CA厂商自己给自己颁发的信任证书，处于信任链的顶端。
直观地来看下什么是根证书，比如我买的是rapidssl的DV证书，它的根证书就是GeoTrust Global CA

<img src="http://blog.chinaunix.net/attachment/201512/22/15790905_1450756001qzoy.png" width="500" height="500" alt="根证书举例"/>

获取JDK?1.6.0_21的信任根证书库：

`cp ${JAVA_HOME}/jre/lib/security/cacerts /tmp/cacerts.bak`

`${JAVA_HOME}/bin/keytool -list -keystore /tmp/cacerts.bak -storepass changeit`

需要注意的是，申请的startssl免费证书是不在这个默认列表里边的哦。想申请免费证书的同学可以省省力气了。


参考:
 
- http://doc.open.alipay.com/doc2/detail.htm?spm=a219a.7386797.0.0.tQKwZj&treeId=58&articleId=103585&docType=1
- https://en.wikipedia.org/wiki/Root_certificate
- https://en.wikipedia.org/wiki/Certificate_authority
- http://blog.csdn.net/fw0124/article/details/48341671
