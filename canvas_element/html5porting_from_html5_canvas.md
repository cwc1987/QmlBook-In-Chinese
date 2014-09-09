# HTML5画布移植（Porting from HTML5 Canvas）

* [https://developer.mozilla.org/en/Canvas_tutorial/Transformations](https://developer.mozilla.org/en/Canvas_tutorial/Transformations)

* [http://en.wikipedia.org/wiki/Spirograph](http://en.wikipedia.org/wiki/Spirograph)

移植一个HTML5画布图像到QML画布非常简单。在成百上千的例子中，我们选择了一个来移植。

**螺旋图形（Spiro Graph）**

我们使用一个来自Mozila项目的螺旋图形例子来作为我们的基础示例。原始的HTML5代码被作为画布教程发布。

下面是我们需要修改的代码：

* Qt Quick要求定义变量使用，所以我们需要添加var的定义：
```
for (var i=0;i<3;i++) {
    ...
}
```

* 修改绘制方法接收Context2D对象：
```
function draw(ctx) {
    ...
}
```

* 由于不同的大小，我们需要对每个螺旋适配转换：
```
ctx.translate(20+j*50,20+i*50);
```

最后我们实现onPaint操作。在onPaint中我们请求一个context，并且调用我们的绘制方法。

```
    onPaint: {
        var ctx = getContext("2d");
        draw(ctx);
    }
```

下面这个结果就是我们使用QML画布移植的螺旋图形。

![](http://qmlbook.org/_images/spirograph.png)

**发光线（Glowing Lines）**

下面有一个更加复杂的移植来自W3C组织。[原始的发光线](http://www.w3.org/TR/2dcontext/#examples)有些很不错的地方，这使得移植更加具有挑战性。

![](http://qmlbook.org/_images/html_glowlines.png)

```
<!DOCTYPE HTML>
<html lang="en">
<head>
    <title>Pretty Glowing Lines</title>
</head>
<body>

<canvas width="800" height="450"></canvas>
<script>
var context = document.getElementsByTagName('canvas')[0].getContext('2d');

// initial start position
var lastX = context.canvas.width * Math.random();
var lastY = context.canvas.height * Math.random();
var hue = 0;

// closure function to draw
// a random bezier curve with random color with a glow effect
function line() {

    context.save();

    // scale with factor 0.9 around the center of canvas
    context.translate(context.canvas.width/2, context.canvas.height/2);
    context.scale(0.9, 0.9);
    context.translate(-context.canvas.width/2, -context.canvas.height/2);

    context.beginPath();
    context.lineWidth = 5 + Math.random() * 10;

    // our start position
    context.moveTo(lastX, lastY);

    // our new end position
    lastX = context.canvas.width * Math.random();
    lastY = context.canvas.height * Math.random();

    // random bezier curve, which ends on lastX, lastY
    context.bezierCurveTo(context.canvas.width * Math.random(),
    context.canvas.height * Math.random(),
    context.canvas.width * Math.random(),
    context.canvas.height * Math.random(),
    lastX, lastY);

    // glow effect
    hue = hue + 10 * Math.random();
    context.strokeStyle = 'hsl(' + hue + ', 50%, 50%)';
    context.shadowColor = 'white';
    context.shadowBlur = 10;
    // stroke the curve
    context.stroke();
    context.restore();
}

// call line function every 50msecs
setInterval(line, 50);

function blank() {
    // makes the background 10% darker on each call
    context.fillStyle = 'rgba(0,0,0,0.1)';
    context.fillRect(0, 0, context.canvas.width, context.canvas.height);
}

// call blank function every 50msecs
setInterval(blank, 40);

</script>
</body>
</html>
```

在HTML5中，context2D对象可以随意在画布上绘制。在QML中，只能在onPaint操作中绘制。在HTML5中，通常调用setInterval使用计时器触发线段的绘制或者清屏。由于QML中不同的操作方法，仅仅只是调用这些函数不能实现我们想要的结果，因为我们需要通过onPaint操作来实现。我们也需要修改颜色的格式。让我们看看需要改变哪些东西。

修改从画布元素开始。为了简单，我们使用画布元素（Canvas）作为我们QML文件的根元素。

```
import QtQuick 2.0

Canvas {
   id: canvas
   width: 800; height: 450

   ...
}
```

代替直接调用的setInterval函数，我们使用两个计时器来请求重新绘制。一个计时器触发间隔较短，允许我们可以执行一些代码。我们无法告诉绘制函数哪个操作是我想触发的，我们为每个操作定义一个布尔标识，当重新绘制请求时，我们请求一个操作并且触发它。

下面是线段绘制的代码，清屏操作类似。

```
...
property bool requestLine: false

Timer {
    id: lineTimer
    interval: 40
    repeat: true
    triggeredOnStart: true
    onTriggered: {
        canvas.requestLine = true
        canvas.requestPaint()
    }
}

Component.onCompleted: {
    lineTimer.start()
}
...
```

现在我们已经有了告诉onPaint操作中我们需要执行哪个操作的指示。当我们进入onPaint处理每个绘制请求时，我们需要提取画布元素中的初始化变量。

```
Canvas {
    ...
    property real hue: 0
    property real lastX: width * Math.random();
    property real lastY: height * Math.random();
    ...
}
```

现在我们的绘制函数应该像这样：

```
onPaint: {
    var context = getContext('2d')
    if(requestLine) {
        line(context)
        requestLine = false
    }
    if(requestBlank) {
        blank(context)
        requestBlank = false
    }
}
```

线段绘制函数提取画布作为一个参数。

```
function line(context) {
    context.save();
    context.translate(canvas.width/2, canvas.height/2);
    context.scale(0.9, 0.9);
    context.translate(-canvas.width/2, -canvas.height/2);
    context.beginPath();
    context.lineWidth = 5 + Math.random() * 10;
    context.moveTo(lastX, lastY);
    lastX = canvas.width * Math.random();
    lastY = canvas.height * Math.random();
    context.bezierCurveTo(canvas.width * Math.random(),
        canvas.height * Math.random(),
        canvas.width * Math.random(),
        canvas.height * Math.random(),
        lastX, lastY);

    hue += Math.random()*0.1
    if(hue > 1.0) {
        hue -= 1
    }
    context.strokeStyle = Qt.hsla(hue, 0.5, 0.5, 1.0);
    // context.shadowColor = 'white';
    // context.shadowBlur = 10;
    context.stroke();
    context.restore();
}
```

最大的变化是使用QML的Qt.rgba()和Qt.hsla()。在QML中需要把变量值适配在0.0到1.0之间。

同样应用在清屏函数中。

```
function blank(context) {
    context.fillStyle = Qt.rgba(0,0,0,0.1)
    context.fillRect(0, 0, canvas.width, canvas.height);
}
```

下面是最终结果（目前没有阴影）类似下面这样。

![](http://qmlbook.org/_images/glowlines.png)

查看下面的链接获得更多的信息：

* [W3C HTML Canvas 2D Context Specification](http://www.w3.org/TR/2dcontext/)

* [Mozilla Canvas Documentation](https://developer.mozilla.org/en/HTML/Canvas)

* [HTML5 Canvas Tutorial](http://www.html5canvastutorials.com/)

