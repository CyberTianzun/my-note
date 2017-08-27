---
title: Android中适配魅族找不到android.settings.WIRELESS_SETTINGS的问题
date: 2015-11-4
tags: "android"
categories: "android"
---
今天遇到这样一个bug，显然魅族的系统中没有这个东西 `android.settings.WIRELESS_SETTINGS`。魅族肯定是自己写了一个，然后把 android 系统原来的那个 Action 给删了。。。你说该界面就复用原来那个不好吗，或者把 Action 加上做得和原来的一样也好啊。

魅族真是太shi了

```
Caused by: android.content.ActivityNotFoundException: No Activity found to handle Intent { act=android.settings.WIRELESS_SETTINGS }
  at android.app.Instrumentation.checkStartActivityResult(Instrumentation.java:1745)
  at android.app.Instrumentation.execStartActivity(Instrumentation.java:1463)
  at android.app.Activity.startActivityForResult(Activity.java:3502)
  at android.app.Activity.startActivityForResult(Activity.java:3463)
  at android.support.v4.app.FragmentActivity.startActivityForResult(FragmentActivity.java:820)
  at cn.hiroz.testaction.MainActivity.onClickButton1(MainActivity.java:41)
```

于是这样解决：

```
if (!android.os.Build.MANUFACTURER.equals("Meizu")) {
    startActivityForResult(new Intent(android.provider.Settings.ACTION_WIRELESS_SETTINGS), 0);
} else {
    startActivityForResult(new Intent(android.provider.Settings.ACTION_WIFI_SETTINGS), 0);
}
```