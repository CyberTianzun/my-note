---
title: Android中遇到HTTPS握手失败SSLHandshakeException的问题
date: 2015-11-5
tags: "android"
categories: "android"
---
我遇到的错误如下
javax.net.ssl.SSLHandshakeException: org.bouncycastle.jce.exception.ExtCertPathValidatorException: Could not validate certificate signature.

## 摘抄分析

最近使用google的Volley框架遇到一个下载图片的异常com.android.volley.error.NoConnectionError: javax.net.ssl.SSLHandshakeException: com.android.org.bouncycastle.jce.exception.ExtCertPathValidatorException: Could not validate certificate: current time: Sat Aug 15 05:14:42 GMT+08:00 1970, validation time: Fri Apr 05 23:15:55 GMT+08:00 2013

通过查看日志发现这个图片的下载地址和其他正常的下载地址不一样，使用的是https协议，但我们产品中以前的图片都是http协议，这几个有问题的图片是UI的妹子随便从某个地方下载的，

然后出现了这个问题，google了一下，这个异常说的是在校验证书的时候出现时间校验失败！打开手机的设置，发现手机上的时间居然是好几年前。。。然后，调整好手机时间即可正常下载。

不过这种解决方式很死板，怎么可能让用户这么作呢！！！

有几位大牛提供了解决方案：

http://my.oschina.net/blackylin/blog/144136

http://www.eoeandroid.com/thread-161747-1-1.html

然后还有一篇关于https协议的好博文:

http://www.cnblogs.com/P_Chou/archive/2010/12/27/https-ssl-certification.html

## 其他的参考文档

http://blog.csdn.net/xhqtyq05251109/article/details/6581431

http://stackoverflow.com/questions/7588082/could-not-validate-certificate-signature

http://www.cnblogs.com/mayongsheng/p/4387109.html

https://github.com/awslabs/aws-sdk-android-samples/issues/26