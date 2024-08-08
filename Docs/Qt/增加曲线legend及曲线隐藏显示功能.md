---
title: 增加曲线legend及曲线隐藏显示功能
layout: default
nav_enabled: true
parent: Qt开发随记
nav_order: 7
---

1. 设置曲线名称
``` c++
    customplot->graph(0)->setName("CurveName");
```
2. 设置legend
``` c++
    // 使能legend
    customPlot->legend->setVisible(true);
    // 设置legend可被点击
    customPlot->legend->setSelectableParts(QCPLegend::spItems);
    // 关联legend被点击及槽函数
    connect(customPlot, SIGNAL(legendClick(QCPLegend*,QCPAbstractLegendItem*,QMouseEvent*)), this, SLOT(onLegendClick(QCPLegend*,QCPAbstractLegendItem*,QMouseEvent*)));
```
3. 点击legend隐藏/显示功能
``` c++
    void RealtimeCurvePlot::onLegendClick(QCPLegend *legend, QCPAbstractLegendItem *item, QMouseEvent *event)
    {
        // 将抽象图例项转换为具体的图例项类型
        QCPPlottableLegendItem *plItem = qobject_cast<QCPPlottableLegendItem*>(item);
        if (plItem)
        {
            // 获取被点击的图例对应的曲线
            QCPGraph* graph = qobject_cast<QCPGraph*>(plItem->plottable());
            if (graph)
            {
                // 切换曲线的可见性
                graph->setVisible(!graph->visible());
    
                // 设置切换可见性曲线对应的legend字体颜色
                plItem->setTextColor(graph->visible() ? Qt::black : Qt::gray);
            }
        }
    }
```
