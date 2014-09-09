# 图片（Images）

QML画布支持多种资源的图片绘制。在画布中使用一个图片需要先加载图片资源。在我们的例子中我们使用Component.onCompleted操作来加载图片。

```
    onPaint: {
        var ctx = getContext("2d")


        // draw an image
        ctx.drawImage('assets/ball.png', 10, 10)

        // store current context setup
        ctx.save()
        ctx.strokeStyle = 'red'
        // create a triangle as clip region
        ctx.beginPath()
        ctx.moveTo(10,10)
        ctx.lineTo(55,10)
        ctx.lineTo(35,55)
        ctx.closePath()
        // translate coordinate system
        ctx.translate(100,0)
        ctx.clip()  // create clip from triangle path
        // draw image with clip applied
        ctx.drawImage('assets/ball.png', 10, 10)
        // draw stroke around path
        ctx.stroke()
        // restore previous setup
        ctx.restore()

    }

    Component.onCompleted: {
        loadImage("assets/ball.png")
    }
```

在左边，足球图片使用10×10的大小绘制在左上方的位置。在右边我们对足球图片进行了裁剪。图片或者轮廓路径都可以使用一个路径来裁剪。裁剪需要定义一个裁剪路径，然后调用clip()函数来实现裁剪。在clip()之前所有的绘制操作都会用来进行裁剪。如果还原了之前的状态或者定义裁剪区域为整个画布时，裁剪是无效的。

![](http://qmlbook.org/_images/canvas_image.png)
