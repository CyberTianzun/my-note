---
title: Eclipse中重命名Android包名出现an unexpected exception occured错误
date: 2015-11-4
tags: "android"
categories: "android"
---
在 Android 中，相同包名的 App 是无法同时安装的。因为临时接到需求，要把我正在做的 App 做一个渠道定制版，又需要可以和在线版本的 App 同时安装。

1. Eclipse 中（老项目伤不起），选中 App 项目点右键弹出菜单，选中 Android Tools -> Rename Application Package

2. 输入希望改成什么包名

3. 理论上这样就完成了

但是我出现了这样的错误：

![rename-application-package-exception](rename-application-package-exception-in-eclipse-1.png)

最后查明真相是这样的：在项目中的java文件被完全注释掉了，就会出现an unexpected exception occured的错误提示，注释掉的文件就应该直接删除嘛，删除文件就解决这个问题了。

用全局搜索 File Search直接搜索 “//package”或者“//public class” 就能很快地找出那些被注释掉的文件了。