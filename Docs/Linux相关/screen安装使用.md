---
title: screen安装使用
layout: default
parent: Linux运维相关
---

# screen安装使用
## Step1 安装
``` bash
sudo apt install screen
```
## 基本使用
1. 新建screen会话
``` bash
screen # 新建并进入绘画
screen -S NAME  # 指令会话名称
```
2. 分离screen会话
使用`Ctrl A, Ctrl D`的组合键（按下ctrl的同时，依次按下A和D）离开screen会话到后台执行，回到主ssh会话
3. 查看screen会话
``` bash
screen -ls
```
4. 回到screen会话
``` bash
screen -r NAME   #使用会话名称
screen -r XXX    #使用会话进程号
```
5. 结束screen会话
回到screen会话后，通过`Ctrl D`，或者`exit`结束当前会话
