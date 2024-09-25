---
title: 右键菜单增加新建draw.io文件
layout: default
parent: 右键菜单相关
grand_parent: Windows系统相关
nav_order: 3
---

# 右键菜单增加新建draw.io文件
## Step 1
1. 运行regedit，打开注册表编辑器
2. 在HKEY_CLASS_ROOT下新建项：.drawio
3. 选中新建的项，在右侧双击"默认"，设置其值为：drawio
4. 右键新建的项.drawio，新建项：ShellNew
5. 右键新建的项ShellNew，新建字符串：NullFile
6. 右键新建的项ShellNew，新建字符串：FileName，修改其值为：C:\Program Files\draw.io\new.drawio（若不执行此项，新建的文件drawio不识别）
7. 在 C:\Program Files\draw.io\ 目录下使用draw.io新建一个文件，命名为new.drawio

## Step 2
1. 在HKEY_CLASS_ROOT下新建项：drawio
2. 选中新建的项，在右侧双击"默认"，设置其值为：drawio文件
3. 重启点计算机即可
