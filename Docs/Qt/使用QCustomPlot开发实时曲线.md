---
title: 使用QCustomPlot开发实时曲线
layout: default
nav_enabled: true
parent: Qt开发随记
---

## 1. 架构设计

使用生产者消费者模型，数据缓冲区使用FIFO

**生产者线程**

负责定期产生一组数据，数据存入缓冲区

**消费者线程**

即曲线显示模块，定时从缓冲区读取一组数据

**FIFO**

具备读写功能，需要确保线程安全

**Note**

***1.消费者线程的读取速率要远大于生产者线程，目的是确保生产一组数据后在最短时间内消费掉，最佳的状态是FIFO中只有一组数据，因此FIFO不用很大***


代码：https://github.com/ledoen/RealtimeCurvePlot
