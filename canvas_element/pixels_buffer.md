# 像素缓冲（Pixels Buffer）

当你使用画布时，你可以检索读取画布上的像素数据，或者操作画布上的像素。读取图像数据使用createImageData(sw,sh)或者getImageData(sx,sy,sw,sh)。这两个函数都会返回一个包含宽度（width），高度（height）和数据（data）的图像数据（ImageData）对象。图像数据包含了一维数组像素数据，使用RGBA格式进行检索。每个数据的数据范围在0到255之间。设置画布的像素数据你可以使用putImageData(imagedata,dx,dy)函数来完成。

另一种检索画布内容的方法是将画布的数据存储进一张图片中。可以使用画布的函数save(path)或者toDataURL(mimeType)来完成，toDataURL(mimeType)会返回一个图片的地址，这个链接可以直接用Image元素来读取。

```
import QtQuick 2.0

Rectangle {
    width: 240; height: 120
    Canvas {
        id: canvas
        x: 10; y: 10
        width: 100; height: 100
        property real hue: 0.0
        onPaint: {
            var ctx = getContext("2d")
            var x = 10 + Math.random(80)*80
            var y = 10 + Math.random(80)*80
            hue += Math.random()*0.1
            if(hue > 1.0) { hue -= 1 }
            ctx.globalAlpha = 0.7
            ctx.fillStyle = Qt.hsla(hue, 0.5, 0.5, 1.0)
            ctx.beginPath()
            ctx.moveTo(x+5,y)
            ctx.arc(x,y, x/10, 0, 360)
            ctx.closePath()
            ctx.fill()
        }
        MouseArea {
            anchors.fill: parent
            onClicked: {
                var url = canvas.toDataURL('image/png')
                print('image url=', url)
                image.source = url
            }
        }
    }

    Image {
        id: image
        x: 130; y: 10
        width: 100; height: 100
    }

    Timer {
        interval: 1000
        running: true
        triggeredOnStart: true
        repeat: true
        onTriggered: canvas.requestPaint()
    }
}
```

在我们这个例子中，我们每秒在左边的画布中绘制一个的圆形。当使用鼠标点击画布内容时，会将内容存储为一个图片链接。在右边将会展示这个存储的图片。

**注意**

**在Qt5的Alpha版本中，检索图像数据似乎不能工作。**
