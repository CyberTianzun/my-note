# Android中使用app_process命令时的建议

## 结论直接写在前面

app_process命令的使用要注意下64位机器的情况，64位机器要用/system/bin/app_process32，强制使用32位模式。

如果我们在app_process起来的进程中加载so的话，有可能会导致加载不起来，我们没32位的so，64位机器上app_process默认是启64位进程。

ref http://blog.csdn.net/jscese/article/details/47101815