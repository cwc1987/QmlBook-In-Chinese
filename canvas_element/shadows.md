# 阴影（Shadows）

**注意**

**在Qt5的alpha版本中，我们使用阴影遇到了一些问题。**

2D对象的路径可以使用阴影增强显示效果。阴影是一个区域的轮廓线使用偏移量，颜色和模糊来实现的。所以你需要指定一个阴影颜色（shadowColor），阴影X轴偏移值（shadowOffsetX），阴影Y轴偏移值（shadowOffsetY）和阴影模糊（shadowBlur）。这些参数的定义都使用2D context来定义。2D context是唯一的绘制操作接口。

阴影也可以用来创建发光的效果。在下面的例子中我们使用白色的光创建了一个“Earth”的文本。在一个黑色的背景上可以有更加好的显示效果。

首先我们绘制黑色背景：

```
        // setup a dark background
        ctx.strokeStyle = "#333"
        ctx.fillRect(0,0,canvas.width,canvas.height);
```

然后定义我们的阴影配置：

```
        ctx.shadowColor = "blue";
        ctx.shadowOffsetX = 2;
        ctx.shadowOffsetY = 2;
        // next line crashes
        // ctx.shadowBlur = 10;
```

最后我们使用加粗的，80像素宽度的Ubuntu字体来绘制“Earth”文本：

```
        ctx.font = 'Bold 80px Ubuntu';
        ctx.fillStyle = "#33a9ff";
        ctx.fillText("Earth",30,180);
```
