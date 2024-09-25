---
title: Typero增加右键菜单
layout: default
parent: 右键菜单相关
grand_parent: Windows系统相关
---
Typero增加右键菜单比较简单，新建一个Typero.reg文件，将以下内容拷贝粘贴进去，打开运行一下，重启电脑即可
``` bash
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT\.md]
@="Typora.md"
"Content Type"="text/markdown"
"PerceivedType"="text"
[HKEY_CLASSES_ROOT\.md\ShellNew]
"NullFile"=""
```
