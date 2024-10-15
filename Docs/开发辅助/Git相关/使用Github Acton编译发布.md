---
title: 使用Github Acton编译发布
layout: default
parent: Git相关
grand_parent: 开发辅助
---

# 使用Github Acton编译发布
# 准备工作
1. 新建token，并存储token内容
2. 新建github仓库
- 新建github仓库
- 在仓库设置页的Actions下
  - Actions permissions 勾选 Allow all actions and reusable workflows
  - Workflow permissions 勾选 Read and write permissions
- 在Secrets下
  - 在Actions下，添加secret，使用之前的token内容
