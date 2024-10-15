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
3. 编写workflow文件
  在仓库根目录下新建`.github/workflows/example.yml`文件，内容如下
```bash
name: GitHub Actions Demo
run-name: ${{ github.actor }} is testing out GitHub Actions 🚀
on:
  repository_dispatch:
  workflow_dispatch:
jobs:
  Explore-GitHub-Actions:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v4
      - run: echo "🎉 The job was automatically triggered by a ${{ github.event_name }} event."
      - name: configure
        run:
          chmod +x configure
          ./configure
      - name: make
        run: make
      - name: test
        run: ./test
      - name: Release
        uses: softprops/action-gh-release@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with: 
          tag_name: prebuild_driver
          body: test demo
          files: test
      - name: clean
        run: make clean
```
4. 菜单栏进入Actions，开启运行，完成后将自动发布编译好文件
