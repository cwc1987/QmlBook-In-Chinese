# 转换（Transformation）

画布有多种方式来转换坐标系。这些操作非常类似于QML元素的转换。你可以通过缩放（scale），旋转（rotate），translate（移动）来转换坐标系。与QML元素的转换不同的是，转换原点通常就是画布原点。例如，从中心点放大一个封闭的路径，你需要先将画布原点移动到整个封闭的路径的中心点上。使用这些转换的方法你可以创建一些更加复杂的转换。

```
// transform.qml

import QtQuick 2.0

Canvas {
    id: root
    width: 240; height: 120
    onPaint: {
        var ctx = getContext("2d")
        ctx.strokeStyle = "blue"
        ctx.lineWidth = 4

        ctx.beginPath()
        ctx.rect(-20, -20, 40, 40)
        ctx.translate(120,60)
        ctx.stroke()

        // draw path now rotated
        ctx.strokeStyle = "green"
        ctx.rotate(Math.PI/4)
        ctx.stroke()
    }
}
```

![](http://qmlbook.org/_images/transform.png)

除了移动画布外，也可以使用scale(x,y)来缩放x,y坐标轴。旋转使用rotate(angle)，angle是角度（360度=2*Math.PI）。使用setTransform(m11,m12,m21,m22,dx,dy)来完成矩阵转换。

**警告**

**QML画布中的转换与HTML5画布中的机制有些不同。不确定这是不是一个Bug。**

**注意**

**重置矩阵你可以调用resetTransform()函数来完成，这个函数会将转换矩阵还原为单位矩阵。**
