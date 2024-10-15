---
title: 添加samba文件共享服务
layout: default
parent: Linux运维相关
---

# 添加samba文件共享服务
## Step1 安装Samba
``` bash
sudo install samba
```
## Step2 配置Samba
修改配置文件
``` bash
sudo nano /etc/samba/smb.conf
```
在文件结尾添加以下内容
``` bash
[share]
  comment = ubuntu share
  path = /home/sharefolder
  browsable = yes
  guest ok = yes
  read only = no
  creat mask = 0755
```
## Step3 创建共享文件夹
如果上述path中的路径不存在，则创建它
## Step4 重启服务
``` bash
sudo systemctl restart smbd
sudo systemctl restart nmbd
```
