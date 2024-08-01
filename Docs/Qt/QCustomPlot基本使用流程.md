---
layout: default
title: QCustomPlot基本使用流程
nav_enabled: true
---
1. 下载qcustomplot.h和qcustomplot.cpp文件，放到项目源文件路径下
2. 修改CMakeLists.txt文件，除了将上述文件加入项目文件列表外，加入以下代码
```c
find_package(Qt${QT_VERSION_MAJOR} REQUIRED PrintSupport)
target_link_libraries(RealtimeCurveV2 PRIVATE Qt${QT_VERSION_MAJOR}::PrintSupport)
```
