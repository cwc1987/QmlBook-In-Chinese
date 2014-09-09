# 渐变（Gradients）

画布中可以使用颜色填充也可以使用渐变或者图像来填充。

```
    onPaint: {
        var ctx = getContext("2d")

        var gradient = ctx.createLinearGradient(100,0,100,200)
        gradient.addColorStop(0, "blue")
        gradient.addColorStop(0.5, "lightsteelblue")
        ctx.fillStyle = gradient
        ctx.fillRect(50,50,100,100)
    }
```

在这个例子中，渐变色定义在开始点（100,0）到结束点（100,200）。在我们画布中是一个中间垂直的线。渐变色在停止点定义一个颜色，范围从0.0到1.0。这里我们使用一个蓝色作为0.0（100,0），一个高亮刚蓝色作为0.5（100,200）。渐变色的定义比我们想要绘制的矩形更大，所以矩形在它定义的范围内对渐变进行了裁剪。

![](http://qmlbook.org/_images/gradient.png)

**注意**

**渐变色是在画布坐标下定义的，而不是在绘制路径相对坐标下定义的。画布中没有相对坐标的概念。**
