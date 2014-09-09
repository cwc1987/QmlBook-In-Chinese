# Canvas Element

**注意**

**最后一次构建：2014年1月20日下午18:00。**

**这章的源代码能够在[assetts folder](http://qmlbook.org/assets)找到。**

![](http://qmlbook.org/_images/glowlines.png)

在早些时候的Qt4中加入QML时，一些开发者讨论如何在QtQuick中绘制一个圆形。类似圆形的问题，一些开发者也对于其它的形状的支持进行了讨论。在QtQuick中没有圆形，只有矩形。在Qt4中，如果你需要一个除了矩形外的形状，你需要使用图片或者使用你自己写的C++圆形元素。

Qt5中引进了画布元素（canvas element），允许脚本绘制。画布元素（canvas element）提供了一个依赖于分辨率的位图画布，你可以使用JavaScript脚本来绘制图形，制作游戏或者其它的动态图像。画布元素（canvas element）是基于HTML5的画布元素来完成的。

画布元素（canvas element）的基本思想是使用一个2D对象来渲染路径。这个2D对象包括了必要的绘图函数，画布元素（canvas element）充当绘制画布。2D对象支持画笔，填充，渐变，文本和绘制路径创建命令。

让我们看看一个简单的路径绘制的例子：

```
import QtQuick 2.0

Canvas {
    id: root
    // canvas size
    width: 200; height: 200
    // handler to override for drawing
    onPaint: {
        // get context to draw with
        var ctx = getContext("2d")
        // setup the stroke
        ctx.lineWidth = 4
        ctx.strokeStyle = "blue"
        // setup the fill
        ctx.fillStyle = "steelblue"
        // begin a new path to draw
        ctx.beginPath()
        // top-left start point
        ctx.moveTo(50,50)
        // upper line
        ctx.lineTo(150,50)
        // right line
        ctx.lineTo(150,150)
        // bottom line
        ctx.lineTo(50,150)
        // left line through path closing
        ctx.closePath()
        // fill using fill style
        ctx.fill()
        // stroke using line width and stroke style
        ctx.stroke()
    }
}
```

这个例子产生了一个在坐标（50,50），高宽为100的填充矩形框，并且使用了画笔来修饰边界。

![](http://qmlbook.org/_images/rectangle.png)

画笔的宽度被设置为4个像素，并且定义strokeStyle（画笔样式）为蓝色。最后的形状由设置填充样式（fillStyle）为steelblue颜色，然后填充完成的。只有调用stroke或者fill函数，创建的路径才会绘制，它们与其它的函数使用是相互独立的。调用stroke或者fill将会绘制当前的路径，创建的路径是不可重用的，只有绘制状态能够被存储和恢复。

在QML中，画布元素（canvas element）充当了绘制的容器。2D绘制对象提供了实际绘制的方法。绘制需要在onPaint事件中完成。

```
Canvas {
    width: 200; height: 200
    onPaint: {
        var ctx = getContext("2d")
        // setup your path
        // fill or/and stroke
    }
}
```

画布自身提供了典型的二维笛卡尔坐标系统，左上角是（0,0）坐标。Y轴坐标轴向下，X轴坐标轴向右。

典型绘制命令调用如下：

1. 装载画笔或者填充模式

2. 创建绘制路径

3. 使用画笔或者填充绘制路径

```
    onPaint: {
        var ctx = getContext("2d")

        // setup the stroke
        ctx.strokeStyle = "red"

        // create a path
        ctx.beginPath()
        ctx.moveTo(50,50)
        ctx.lineTo(150,50)

        // stroke path
        ctx.stroke()
    }
```


这将产生一个从P1（50，50）到P2（150,50）水平线。

![](http://qmlbook.org/_images/line.png)

**注意**

**通常在你重置了路径后你将会设置一个开始点，所以，在beginPath()这个操作后，你需要使用moveTo来设置开始点。**
