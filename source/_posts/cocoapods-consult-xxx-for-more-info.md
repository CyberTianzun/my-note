---
title: 使用 Cocoapods 总是提示 There was an error reading CocoaPods-version.yml 错误
date: 2015-11-4
tags: ["mac", "ios"]
categories: "ios"
---
今天在使用 `pod install` 的时候总是提示失败（错误如下）。

啰嗦一下就是cocoapods发布的时候很明显没有检查依赖更新，也没有明确指定依赖库的版本。如果依赖库每次升级都是兼容性的，那就没有问题，如果依赖库不是兼容性升级的，那你的项目也没法使用了。

```
$ pod install

Setting up CocoaPods master repo
[!] There was an error reading '/Users/hiro/.cocoapods/repos/master/CocoaPods-version.yml'.
Please consult http://blog.cocoapods.org/Repairing-Our-Broken-Specs-Repository/ for more information.
```

按照提示我到 [Repairing Our Broken Specs Repository](http://blog.cocoapods.org/Repairing-Our-Broken-Specs-Repository/) 去获取更多信息，但是官网的解决办法：

```
$ sudo rm -fr ~/.cocoapods/repos/master
$ pod setup
```

实际上仍然出错：

```
[ ~ ] ✔ pod setup
Setting up CocoaPods master repo
[!] There was an error reading '/Users/hiro/.cocoapods/repos/master/CocoaPods-version.yml'.
Please consult http://blog.cocoapods.org/Repairing-Our-Broken-Specs-Repository/ for more information.
```

## 解决办法

首先我尝试了清楚一下 gem

```
sudo gem uninstall ocunit2junit
sudo gem uninstall sinatra
sudo gem cleanup
```

实际上并没有什么卵用。这个错误是因为 `CocoaPods-version.yml` 于是把读这个文件的依赖 `psych` 重新装一下。

```
sudo gem uninstall psych
sudo gem install psych -v 2.0.0
sudo rm -fr ~/.cocoapods/repos/master
sudo gem uninstall cocoapods
sudo gem update
sudo gem install cocoapods
pod setup
```

好了可以用了！

## 参考文献

[官方的Issues](https://github.com/CocoaPods/CocoaPods/issues/2908)