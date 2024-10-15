---
title: 安装openssh-server
layout: default
parent: linux运维相关
---

# 安装openssh-server
## Step1 安装
``` bash
sudo apt update
sudo apt install openssh-server
```
安装完成后可以查看状态
``` bash
sudo systemctl status ssh
```
## Step2 设置防火墙（可选）
如果客户端连不上，需要设置防护墙
```bash
sudo ufw allow ssh
```
