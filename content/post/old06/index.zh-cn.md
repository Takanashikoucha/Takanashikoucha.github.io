---
title: "获取Q群成员的邮箱的完成后记录"
slug: ""
description: "和标题好像没啥关系"
date: 2023-06-23T02:56:17+08:00
lastmod: 2023-06-23T02:56:17+08:00
draft: false
toc: true
weight: false
image: ""
categories: ["old"]
tags: ["old"]
---
上周,和以前物理竞赛的小伙伴[现在在武大],合作了一波,帮忙给思课写了个小工具,分享一下一些心得.

tkinter监听窗口关闭事件的方法:

```
root.protocol("WM_DELETE_WINDOW", quit)
# 其中第二个参数是需要执行的函数
```

需要结束进程树的时候:
```
os.system('taskkill /f /t /im *******')
调用结束进程树的命令就好了[写的工具存在一些谜一样的无法kill进程问题]
```

推荐使用subprocess模块替代os.system命令

无法打包情况下的最差方案:
```
python32.exe /quiet PrependPath=1
@echo "python install OK"
@echo "         "
@echo "         "
@echo "         "
@echo "try to get depend"
python .\depend.py
------------------------------------
第一行静默后台安装python32位并且加入path
后面则是执行python的依赖安装,其实可以直接写python -m install ***
```

如何隐藏控制台窗口启动程序:
```
@echo off 
if "%1" == "h" goto begin 
mshta vbscript:createobject("wscript.shell").run("%~nx0 h",0)(window.close)&&exit 
:begin
下面是你自己的代码。
```

----------
给我来瓶冰阔落？

![Code](alipay.jpg)