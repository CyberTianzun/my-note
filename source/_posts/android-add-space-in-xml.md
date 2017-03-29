---
title: Android中的string.xml里如何给字符串前后加空格
date: 2015-11-4
tags: "android"
categories: "android"
---
咱们360手机助手在创建桌面图标“我的游戏”的之后发现，有一款叫做“九游”的机智App把我们的桌面图标“我的游戏”给删除了。我们对其进行了逆向工程，我们从九游的代码看到，九游软件会从桌面数据库查询“我的游戏”关键字。而且是专门针对htc手机做了适配。如果查到了，就删除所有名字叫“我的游戏”的快捷方式。要改这个问题，只能修改我们的快捷方式的名字，或者联系“九游”修改代码。

很明显，机智的我决定在快捷方式的名字前后加空格。但是用 `" " + getString(R.string.xxx) + " "` 这种代码写起来太low，我决定把空格加在string.xml里边。

## 解决方案

如果输入 html 里边的 `&nbsp;` 会出现下面这个错误：

```
Error:The entity "nbsp" was referenced, but not declared.
```

原来在 android 的 xml 定义里边，是不支持这种写法的，那怎么办呢。

我搜索了一下大谷歌，发现有在 android 的 xml 里边有下面两种替代办法。

1. `&#160;`
2. `\u0020`

这两个都代表了空格，并且是有效的。

## 引申

在查阅资料的时候，我发现了一些关于使用 `&#160;` 和 `&#032;` 的一些说法

1. This doesnt work with wrap_content of a textview, when the space occurs at the end of the text. The textview does not maintain the width that is taken up by the 'space' character. –  toobsco42 Nov 23 '13 at 4:13

2. This will add a non-breakable space. It is different from a regular space and will not work well with certain functions (XPath's normalize-space, for example). Try using &#032; (regular space) instead of &#160;. @toobsco42 –  shwartz Feb 10 '14 at 16:43