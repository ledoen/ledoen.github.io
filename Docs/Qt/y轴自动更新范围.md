---
layout: default
title: y轴自动更新范围
nav_enabled: true
parent: Qt开发随记
nav_order: 3
---

代码
  ```c++
  m_pPlotWidget->yAxis->rescale(true);//自动缩放
  m_pPlotWidget->replot();
  ```
