---
layout: default
title: QCustomPlot使用OpenGL
nav_enabled: true
parent: Qt开发随记
nav_order: 6
---

1. 修改项目CMakeLists.txt文件，加入：
``` c
    find_package(Qt6 REQUIRED COMPONENTS OpenGL OpenGLWidgets)
    target_link_libraries(mytarget PRIVATE Qt6::OpenGL)
    target_link_libraries(RealtimeCurveV2 PRIVATE Qt6::OpenGLWidgets)
```

2. 修改qcustomplot.h文件，加入
``` c++
    #include <QOpenGLWidget>
    #define QCUSTOMPLOT_USE_OPENGL
```

3. 此时编译会报错，定位到报错的位置，修改qcustomplot.h文件以下内容（第930行左右）
``` c++
    glClearColor(color.redF(), color.greenF(), color.blueF(), color.alphaF());
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```
为
``` c++
    context.data()->functions()->glClearColor(color.redF(), color.greenF(), color.blueF(), color.alphaF());
    context.data()->functions()->glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

4. 打开qcustomplot中的opengl。输入以下语句
``` c++
    customPlot->setOpenGl(true);
```
