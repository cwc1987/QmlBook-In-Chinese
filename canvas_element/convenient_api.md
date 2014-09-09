# 便捷的接口（Convenient API）

在绘制矩形时，我们提供了一个便捷的接口，而不需要调用stroke或者fill来完成。

```
// convenient.qml

import QtQuick 2.0

Canvas {
    id: root
    width: 120; height: 120
    onPaint: {
        var ctx = getContext("2d")
        ctx.fillStyle = 'green'
        ctx.strokeStyle = "blue"
        ctx.lineWidth = 4

        // draw a filles rectangle
        ctx.fillRect(20, 20, 80, 80)
        // cut our an inner rectangle
        ctx.clearRect(30,30, 60, 60)
        // stroke a border from top-left to
        // inner center of the larger rectangle
        ctx.strokeRect(20,20, 40, 40)
    }
}
```

![](http://qmlbook.org/_images/convenient.png)

**注意**

**画笔的绘制区域由中间向两边延展。一个宽度为4像素的画笔将会在绘制路径的里面绘制2个像素，外面绘制2个像素。**
