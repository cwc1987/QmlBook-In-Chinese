# 画布绘制（Canvas Paint）

在这个例子中我们将使用画布（Canvas）创建一个简单的绘制程序。

![](http://qmlbook.org/_images/canvaspaint.png)

在我们场景的顶部我们使用行定位器排列四个方形的颜色块。一个颜色块是一个简单的矩形，使用鼠标区域来检测点击。

```
    Row {
        id: colorTools
        anchors {
            horizontalCenter: parent.horizontalCenter
            top: parent.top
            topMargin: 8
        }
        property variant activeSquare: red
        property color paintColor: "#33B5E5"
        spacing: 4
        Repeater {
            model: ["#33B5E5", "#99CC00", "#FFBB33", "#FF4444"]
            ColorSquare {
                id: red
                color: modelData
                active: parent.paintColor == color
                onClicked: {
                    parent.paintColor = color
                }
            }
        }
    }
```

颜色存储在一个数组中，作为绘制颜色使用。当用户点击一个矩形时，矩形内的颜色被设置为colorTools的paintColor属性。

为了在画布上跟踪鼠标事件，我们使用鼠标区域（MouseArea）覆盖画布元素，并连接点击和移动操作。

```
    Canvas {
        id: canvas
        anchors {
            left: parent.left
            right: parent.right
            top: colorTools.bottom
            bottom: parent.bottom
            margins: 8
        }
        property real lastX
        property real lastY
        property color color: colorTools.paintColor

        onPaint: {
            var ctx = getContext('2d')
            ctx.lineWidth = 1.5
            ctx.strokeStyle = canvas.color
            ctx.beginPath()
            ctx.moveTo(lastX, lastY)
            lastX = area.mouseX
            lastY = area.mouseY
            ctx.lineTo(lastX, lastY)
            ctx.stroke()
        }
        MouseArea {
            id: area
            anchors.fill: parent
            onPressed: {
                canvas.lastX = mouseX
                canvas.lastY = mouseY
            }
            onPositionChanged: {
                canvas.requestPaint()
            }
        }
    }
```

鼠标点击存储在laxstX与lastY属性中。每次鼠标位置的改变会触发画布的重绘，这将会调用onPaint操作。

最后绘制用户的笔划，在onPaint操作中，我们绘制从最近改变的点上开始绘制一条新的路径，然后我们从鼠标区域采集新的点，使用选择的颜色绘制线段到新的点上。鼠标位置被存储为新改变的位置。
