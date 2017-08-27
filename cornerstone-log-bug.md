---
title: Mac下Cornerstone无法查看SVN日志的问题的解决办法
date: 2015-11-4
tags: ["mac", "version-control"]
categories: "mac"
---
在 Cornerstone 中点击 Log 总是提示 “Could not contact repository to read the latest log entries”。

在 Stackoverflow 上找到了这个

I was having the same problem and emailed Zennaware's Cornerstone support. They sent me these instructions that seem to have fixed the problem for me:

1. Quit Cornerstone
2. Open Terminal
3. Copy paste the following line into Terminal:

defaults delete com.zennaware.Cornerstone HistoryCacheUsage

4. Press return
5. Open Finder
6. Select Go->Go To Folder…
7. Enter:

~/Library/Caches/Cornerstone

and click “Go”
8. Move the selected directory (“Cornerstone”) to the Trash.
9. Restart Cornerstone

The next time Cornerstone prompts you to cache the repository log, choose “Never”.

测试成功！